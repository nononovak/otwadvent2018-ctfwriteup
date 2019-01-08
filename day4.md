# Grille_me - Crypto/net (150)

How well do you know classic ciphers? Some coding may be required...

PEE151R4HEL.LCC8.2TFECE.EOT.901OTHN.ANT23P2RRAG.SNO0.O0TULE.

## Part One

The encoding given above is only the first part of the challenge. After staring at it for a while, I started to notice that the numbers given were the same as the IP address used in previous challenges along with `1204`, the month-day convention for ports previously used. With that it wasn't too hard to see that the characters could just be arranged in a grid and the first message read off the columns:

```
PEE151R4HEL.
LCC8.2TFECE.
EOT.901OTHN.
ANT23P2RRAG.
SNO0.O0TULE.

pleaseconnectto18.205.93.120port1204forthetruechallenge
```

## Part Two

The second part of the challenge comes when you connect to the service above. All you're given is a jumble of characters followed by a prompt. If you enter something wrong, the only response is then 'Invalid password'. After requesting several times, it becomes clear that some of the characters stay constant. A selection of different jumbles is shown below.

```
YPWI2ZORASODSW4E9USR:9N
YPWI2TORASODS6F3DUSR:U6
YPWI1QORASODSGNX3USR:KZ
YPWI46ORASODSL2KAUSR:WH
YPWIDGORASODSDSLKUSR:YK
YPWIX6ORASODS4DPHUSR:0A
YPWIXJORASODSY4Q8USR:BB
YPWIJHORASODSIQDWUSR:1N
YPWIOXORASODSMSZDUSR:C2
YPWIR3ORASODSWHTSUSR:HB
YPWI83ORASODS543RUSR:HF
YPWI70ORASODS8DJ3USR:81
YPWIWIORASODSLNCDUSR:NH
YPWIT1ORASODSOGKVUSR:Z5
YPWIJ7ORASODSLMHYUSR:CG
YPWI9XORASODSGNV3USR:02
YPWIC3ORASODSKXHTUSR:YF
YPWIIUORASODSQVXSUSR:0F
```

From here, I isolated the characters that were constant (`YPWIORASODSUSR:`), and anagramed them to what seems like the logical phrase `YOURPASSWORDIS:`. At this point, I was hopelessly lost on how to un-jumble the remaining 8 characters. Since there were only 8 characters, I chose the brute-force approach and wrote a script to attempt all `8! = 40320` possibilities.

```python
if __name__ == '__main__':
	invalid_ids = []
	ans = b''
	for index in range(40320):
		try:
			print(index)
			positions = [4,5,13,14,15,16,21,22]
			s = socket.socket()
			s.settimeout(1)
			s.connect(('18.205.93.120',1204))
			val = s.recv(25)
			count = 0
			while len(val) != 25 and count < 5:
				count += 1
				val = val + s.recv(25-len(val))
				time.sleep(1)
			if count >= 10:
				raise Exception('recv error')
			if len(val) != 25:
				print('error, got:', val)
				break
			pw = []
			x = index
			ans_positions = []
			while len(positions) > 0:
				i = x % len(positions)
				x = x // len(positions)
				pw.append(val[positions[i]])
				ans_positions.append(positions[i])
				positions = positions[:i] + positions[i+1:]

			s.send(bytes(pw)+b'\n')
			res = s.recv(200)
			print(val,bytes(pw),res)
			if not res.startswith(b'Invalid'):
				print(ans_positions)
				break
			s.close()
		except Exception as e:
			print(e)
			invalid_ids.append(index)
			print('invalid_ids =', invalid_ids)
			time.sleep(5)
	print('invalid ids:', invalid_ids)
```

With this script running, I went to sleep and awoke to find that it hit on the ordering `[13, 4, 14, 21, 15, 5, 16, 22]`.

With the benefit of retrospect, it seems a little more obvious what the actual ordering was. With the numbers arranged in three rows, the order simply reads off as a diagonal pattern as shown below:

```
0, 6 or 10, 17, 7 or 19, 1, 8, 9 or 12 or 18, 9 or 12 or 18, 2, 6 or 10, 7 or 19, 11, 3, 9 or 12 or 18, 20, 13, 4, 14, 21, 15, 5, 16, 22

0 . . . 1 . . . 2 . . . 3 . . . 4 . . . 5 . .
. 6 . 7 . 8 . 9 .10 .11 .12 .13 .14 .15 .16 .
. .17 . . .18 . . .19 . . .20 . . .21 . . .22
```

## Part Three

After entering the correct password to part two, we're greeted with something we expected all along - a [Grille Cipher](https://en.wikipedia.org/wiki/Grille_(cryptography)) (see the challenge name). The response we're given looks like the following, but changes on every new iteration:

```
### # 
#### #
## ###
 ### #
### ##
# ## #

EPEPAL
RUS9E2
OSAOOW
SNBOEE
MROER8
DN:WTE
```

It turns out that this is a similar sentence, but the "password" this time is all contained in the last rotation of the grille. Knowing this, I wrote a quick function to solve the cipher and submitting the answer gives the flag.

```python
def solve_chart(chart):
	print(chart.decode('ascii'))
	arr = chart.split(b'\n')
	pw = []
	for i in range(6):
		for j in range(6):
			print(i,j,chr(arr[j][5-i]),chr(arr[7+i][j]))
			if arr[j][5-i] == ord(' '):
				pw.append(arr[7+i][j])
		print()
	return bytes(pw)+b'\n'
```

And my program output:

```
0 0   E
0 1 # P
0 2 # E
0 3 # P
0 4 # A
0 5 # L

1 0 # R
1 1   U
1 2 # S
1 3   9
1 4 # E
1 5   2

2 0   O
2 1 # S
2 2 # A
2 3 # O
2 4   O
2 5 # W

3 0 # S
3 1 # N
3 2   B
3 3 # O
3 4 # E
3 5 # E

4 0 # M
4 1 # R
4 2 # O
4 3 # E
4 4 # R
4 5   8

5 0 # D
5 1 # N
5 2 # :
5 3   W
5 4 # T
5 5 # E

b'EU92OOB8W\n'
b'Have a flag: AOTW{m33t_m3_at_grillby_s}\n'
```
