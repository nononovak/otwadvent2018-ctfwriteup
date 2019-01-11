# Playagame - Net (150)

Global Thermonuclear War, Cleanup Edition

Service: telnet 18.205.93.120 1203

## Challenge Setup

This challenge wasn't particularly difficult, but did require quite a bit of programming to get to a flag. First off, the service starts with a long menu

```
GREETINGS PROFESSOR FALKEN.

SHALL WE PLAY A GAME?

    TIC-TAC-TOE
    BLACK JACK
    GIN RUMMY
    HEARTS
    BRIDGE
    CHECKERS
    CHESS
    POKER
    FIGHTER COMBAT
    GUERRILLA ENGAGEMENT
    DESERT WARFARE
    AIR-TO-GROUND ACTIONS
    THEATERWIDE TACTICAL WARFARE
    THEATERWIDE BIOTOXIC AND CHEMICAL WARFARE

    GLOBAL THERMONUCLEAR WAR, CLEANUP EDITION
```

after which you're expected to input `GLOBAL THERMONUCLEAR WAR, CLEANUP EDITION`. The game rules and setup then follow and are pretty self-explanatory

```
In 2025, the world has been devastated by a nuclear attack and is a nuclear
wasteland. Luckily, the Chinese government has foreseen this disaster and
launched a satellite in orbit that is able to counter the nuclear waste.

Research has shown that nuclear waste is primarily composed of inverted tachyon
and positive lepton radiation, which cancel each other out.

The Chinese satellite uses inverted tachyons (-) and positive leptons (+) that
can each be fired in 9 different patterns over an area of 3 by 3 kilometers.

Your job is to clean up the nuclear waste by positioning the satellite,
selecting a firing mode (- or +) and a firing pattern (1 through 9), and then
firing.  The result will be instantaneous, the remaining contaminants per
square kilometer will be indicated on the screen.

Please note that firing the satellite costs money, which the post-apocalyptic
world is in short supply of: don't waste money! Good luck!

Keys:
        arrow keys: position satellite
        s or c: select firing mode and pattern
        f: fire satellite
        q: quit and let the world go to waste


Press enter to continue.

```

Following this you get a game grid which is colored, but looks like this underneath the ANSI coloring.

```
 0  0  0  0  0 -1  4  6  18 20 18 6  4 -3  0  0  0  0  0  0  0  0  0  0  0 
 0  0  0  0  0  2 -2  16 19 17 18 7  7  4  0  0  0  0  0  0  0  0  0  0  0 
 0  0  0  0  0 -1  12 18 20 19 20 18 8 -2  0  0  0  0  0  0  0  0  0  0  0 
 0  0  0  0  0  0 -5  9 -9  12 4  17-4  0  0  0  0  0  0  0  0  0  0  0  0 
 0  0  0  0  0  0  0  0  8  14 17 12 5  0  0  0  0  0  0  0  0  0  0  0  0 
 0  0  0  0  0 -3  5 -3 -1  6 -7  6 -3  0  0  0  0  0  0  0  0  0  0  0  0 
 0  0 -1 -1  5  3  11 20 7  4  0  0  0  0  0  0  0  0  0  0  0 -3  5 -1 -1 
 0  0  0  13 12 6  6  14 8 -3  0  0  0  0  0  0  0  0  0  0 -2  10 11 12 1 
 0 -3  9  5  16 11 6  19 7  2  0  0  0  0  0  0  0  0  0  0  4  3  12 5  2 
 0  6  8  13 8  20 5  6 -1 -1  0  0  0  0  0  0  0  0  0  0 -2  7  12 20 18
 0 -5  14 5  17 17 7  1  0  0 -2  1  3 -1 -1  0  0  0  0  0  0  0  4  20 12
 0  4  6  8  4  3  0 -4  5 -2  4  14 19 9 -2  7 -2 -1  0  0  0 -1  7  10 11
 0 -2  4 -3  2 -1  0  5  15 11 7  9  10 0  7  17 15 0  0  0  0  0 -3  6 -3 
-2  4 -2  0  0  0  0 -2  13 5  18 14 15 7  5  20 7  2  0  0  0  0  0  0  0 
 4  6  4  0  0  0  0  1  5  5  9  4  1  3  14 15 17-2  0  0  0  0  0  0  0 
 13 10-4  0  0  0  0 -1  2 -3  4 -2  0 -2  0  5  14-1  0  0  0  0  0  0  0 
 18 10 4  0  0  0  0  0  0  0  0  0  0 -3  4  15 18 6  0  0  0  0  0  0  0 
 3 -1 -3  3  1 -2  0  0  0  0  0  0  0  6  19 20 15-4  0  0  0  0  0  0  0 
 7  9  11 14 18 2  0  0  0  0  0 -1  2 -4  2  5 -4  0  0  0  0  0 -3  6 -3 
 11 19 12 12 8  3 -1  0  0  0 -2  4  5  2 -2 -1  2  3 -3  0  0  0  5  10 12
 4  20 19 9  15 5  1  1 -1  0  1  16 14 8 -1  17 17 19 5  0  0  0 -2  18 19
 4  8  8  11 9  20 4  13-2  0  4  16 13 6  8  16 17 10-2  0  0  0  1  6  18
 20 18 19 2  15 11 19 19 6  0 -3  5 -2  1 -5  9  3  7  1  0  0  0 -3  2  4 
 9  0  19 15 18 20 7  15-2  0  0  0  0  0  0 -1  1  1 -1  0  0  0  4  12 10
 11 3  16 1  9 -2  2  0 -1  0  0  0  0  0  0  0  0  0  0  0  0  0 -3  4  1 

Current mode: +1     Money spent:     0.00 MM¥      Contaminants remaining: 2584

Last result: unknown command
Your move: 
```

Clearly this grid is large enough that we can't just manually solve it for an answer, and furthermore we don't know what each "firing pattern" corresponds to. Also, there the setup also makes some mention of "don't waste money!" by firing too many shots. I'm not sure I ever ran into this constraint with my final solution, but it was something I considered but then forgot about in my solution.

Taking these issues one at a time, the first was to identify each "firing pattern". It turns out these are the same across each new instance of the game (thank you!), but the game board itself is randomly generated each time you connect. Ok, so it was easy enough to move the cursor, fire each pattern, and record the firing pattern as follows:

```
patterns s-9
 -2  0  2
  1  0  1
  0  1  1
patterns s-8
  2  1 -2
  0  0 -1
  0 -2 -1
patterns s-7
 -2 -1  2
  0  0  1
  1  2  1
patterns s-6
  4  0 -5
 -2  1 -2
 -1 -2 -2
patterns s-5
  3  0 -3
 -1  1 -2
 -1 -1 -1
patterns s-4
 -1  0  1
  0  1  0
  1  1  1
patterns s-3
  1 -1 -2
 -1  1 -1
  1  1  0
patterns s-2
 -1  0  5
  2 -3  1
 -1  0  1
patterns s-1
 -5  0  5
  1 -1  2
  1  2  2
patterns s+1
  5  0 -5
 -1  1 -2
 -1 -2 -2
patterns s+2
  1  0 -5
 -2  3 -1
  1  0 -1
patterns s+3
 -1  1  2
  1 -1  1
 -1 -1  0
patterns s+4
  1  0 -1
  0 -1  0
 -1 -1 -1
patterns s+5
 -3  0  3
  1 -1  2
  1  1  1
patterns s+6
 -4  0  5
  2 -1  2
  1  2  2
patterns s+7
  2  1 -2
  0  0 -1
 -1 -2 -1
patterns s+8
 -2 -1  2
  0  0  1
  0  2  1
patterns s+9
  2  0 -2
 -1  0 -1
  0 -1 -1
```

Using this I wrote up a small script to pair each of these up and see if there were any pairs of patterns which maximized the number of unaffected (0) positions. Helpfully, there were a couple pairs I found useful along with a couple notes.

```
patterns s+9 s+8
  0 -1  0
 -1  0  0
  0  1  0
patterns s-9 s-8  ******2.  1 (upper right), 3 (top row)
  0  1  0
  1  0  0
  0 -1  0
patterns s+9 s-7
  0 -1  0
 -1  0  0
  1  1  0
patterns s+8 s+7
  0  0  0
  0  0  0
 -1  0  0
patterns s+6 s+1
  1  0  0
  1  0  0
  0  0  0
patterns s-8 s-7   ******3
  0  0  0
  0  0  0
  1  0  0
patterns s-8 s+3
  1  2  0
  1 -1  0
 -1 -3 -1
patterns s-7 s-3  ******1.   2 (right column)
 -1 -2  0
 -1  1  0
  2  3  1
patterns s-9 s-3
 -1 -1  0
  0  1  0
  1  2  1
patterns s-7 s+4
 -1 -1  1
  0 -1  1
  0  1  0
patterns s-9 s+4
 -1  0  1
  1 -1  1
 -1  0  0
patterns s-6 s-2
  3  0  0
  0 -2 -1
 -2 -2 -1
patterns s-2 s+1
  4  0  0
  1 -2 -1
 -2 -2 -1
```

If it isn't clear, I was looking for a pattern which would only have one non-zero cell on a side or sides of the pair-pattern. Also, I should mention that if you fired on an edge, then nothing happened to the gameboard for the squares hanging over the edge (and off the gameboard).

## Writing a Solver

From here, I set about writing a (quite long) solver to iteratively set cells on the gameboard to zero. I started on the right-most column and used a brute force pattern to move down the column and set it to zeros with various patterns (`s-6`,`s-5`,`s-3`,`s+4`,`s-6 and s-2` and their inverses). With the rightmost column cleared, I moved across the board, using increasingly restrictive patterns to clear individual cells until I needed the `s-8 s-7` pair to clear a single cell at the end.

The full python code I wrote is given below, and can (hopefully) explain my solution much better than any description I give

```python
import asyncio, telnetlib3
import re
from collections import defaultdict
import time
import sys

# http://ascii-table.com/ansi-escape-sequences.php
arrow_right = '\x1b[C'
arrow_down = '\x1b[B'
arrow_left = '\x1b[D'
arrow_up = '\x1b[A'

def chunk_ansi(data):
	arr = []
	last_end = 0
	pat = re.compile('\x1b\[([0-9;]*)(\w)')
	for match in re.finditer(pat,data):
		start = match.start()
		end = match.end()
		if last_end != start:
			arr.append(data[last_end:start])
		if match.group(2) == 'H':
			arr.append(tuple(int(x) for x in match.group(1).split(';')))
		last_end = end
	if last_end != len(data):
		arr.append(data[last_end:])
	return arr

def text_from_ansi(arr):
	return ''.join([x for x in arr if type(x) == str])

def is_move_prompt(buf):
	if buf.endswith('\r\n\nYour move: '):
		return True
	if buf.endswith('\r\n\nLast result: None\r\nYour move: '):
		return True
	if buf.endswith('\r\nYour move: ') and 'moved' not in buf[-50:]: # somehow "moved to ..." is always followed by another grid
		return True
	return False

def read_grid(buf):
	grid = defaultdict(lambda:0)
	pat_line_str = '([0-9\- ]{3})' * 25 + '\r\n'
	pat_grid_str = pat_line_str * 25
	pat_grid = re.compile(pat_grid_str)

	match = re.search(pat_grid,buf)
	if match is None:
		return grid
	for x in range(25):
		for y in range(25):
			grid[x,y] = int(match.group(25*y+x+1).strip())
	return grid

def read_grid_arr(arr):
	grid = defaultdict(lambda:0)
	for index in range(len(arr)):
		if arr[index] == (0,0) and arr[index+1] == '\n':
			for y in range(25):
				for x in range(25):
					grid[x,y] = int(arr[index+2+26*y+x].strip())
	return grid

def deduce_pattern(last_grid,grid,pos):
	h = {}
	posx,posy=pos
	for x in range(-1,2,1):
		for y in range(-1,2,1):
			h[x,y] = grid[posx+x,posy+y]-last_grid[posx+x,posy+y]
	return h

@asyncio.coroutine
def shell(reader, writer):
	full_outp = ''
	moves = [arrow_right]
	#moves = [arrow_right,arrow_down,'s-9','f','s-8','f','s-7','f','s-6','f','s-5','f','s-4','f','s-3','f','s-2','f','s-1','f','s+1','f','s+2','f','s+3','f','s+4','f','s+5','f','s+6','f','s+7','f','s+8','f','s+9','f',arrow_left,arrow_up]
	#moves.append('q') # for debugging
	pos = (0,0)
	fire_pattern = None
	last_move = None
	pattern_hash = {}
	last_grid = None
	double_pattern = None
	while True:
		outp = yield from reader.read(1024)
		if not outp:
			# EOF
			return
		full_outp = full_outp + outp

		arr = chunk_ansi(full_outp)
		buf = text_from_ansi(arr)

		print(arr, flush=True)
		print(buf.encode('utf-8'), flush=True)

		if buf.endswith('GLOBAL THERMONUCLEAR WAR, CLEANUP EDITION'):
			print(buf, flush=True)
			print('writing: GLOBAL THERMONUCLEAR WAR, CLEANUP EDITION')
			writer.write('GLOBAL THERMONUCLEAR WAR, CLEANUP EDITION\n')
			full_outp = ''

		if buf.endswith('Press enter to continue.\n\n'):
			print(buf, flush=True)
			print('writing: \\n')
			writer.write('\n')
			full_outp = ''

		if is_move_prompt(buf) and len(moves) == 0:
			print(buf, flush=True)
			grid = read_grid(buf)
			grid = read_grid_arr(arr)
			x = 24
			for y in range(0,25,1):
				if grid[x,y] != 0:
					break
			if grid[x,y] == 0:
				y = 0
				for x in range(24,-1,-1):
					if grid[x,y] != 0:
						break
			if grid[x,y] == 0:
				for x in range(24,-1,-1):
					for y in range(0,25,1):
						if grid[x,y] != 0:
							break
					if grid[x,y] != 0:
						break

			movetox,movetoy = None,None
			print('working on position',(x,y),'which has value',grid[x,y])
			#if y != 24 and x > 0:
			if y != 24 and x == 24 or (y == 0 and x > 0):
				movetox,movetoy = (x-1,y+1)
				if grid[x,y] >= 5:
					moves.append('s-6')
				elif grid[x,y] >= 3:
					moves.append('s-5')
				elif grid[x,y] >= 2:
					moves.append('s-3')
				elif grid[x,y] >= 1:
					moves.append('s+4')
				elif grid[x,y] <= -5:
					moves.append('s+6')
				elif grid[x,y] <= -3:
					moves.append('s+5')
				elif grid[x,y] <= -2:
					moves.append('s+3')
				elif grid[x,y] <= -1:
					moves.append('s-4')
				moves.append('f')
			#elif y == 24 and x > 0:
			elif y == 24 and x == 24:
				movetox,movetoy = (x-1,y)
				'''
				patterns s-6 s-2
				  3  0  0
				  0 -2 -1
				 -2 -2 -1
				patterns s-9 s-3
				 -1 -1  0
				  0  1  0
				  1  2  1
  				patterns s-8 s+3
				  1  2  0
				  1 -1  0
				 -1 -3 -1
				'''
				if grid[x,y] >= 1:
					moves.append('s-6')
					moves.append('f')
					moves.append('s-2')
					moves.append('f')
				elif grid[x,y] <= -1:
					moves.append('s+6')
					moves.append('f')
					moves.append('s+2')
					moves.append('f')
			#elif x == 0 and y != 24:
			elif x == 0 and y == 0:
				movetox,movetoy = (x,y+1)
				'''
				patterns s-9 s-8
				  0  1  0
				  1  0  0
				  0 -1  0
				'''
				if grid[x,y] >= 1:
					moves.append('s+9')
					moves.append('f')
					moves.append('s+8')
					moves.append('f')
				elif grid[x,y] <= -1:
					moves.append('s-9')
					moves.append('f')
					moves.append('s-8')
					moves.append('f')
			else:
				movetox,movetoy = x+1,y-1
				'''
				patterns s-8 s-7
				  0  0  0
				  0  0  0
				  1  0  0
				'''
				if grid[x,y] >= 1:
					n = grid[x,y]
					moves.append('s+8')
					moves = moves + ['f'] * n
					moves.append('s+7')
					moves = moves + ['f'] * n
				elif grid[x,y] <= -1:
					n = -1 * grid[x,y]
					moves.append('s-8')
					moves = moves + ['f'] * n
					moves.append('s-7')
					moves = moves + ['f'] * n

			# navigate to x+1,y+1
			posx,posy = pos
			if posx > movetox:
				moves = [arrow_left] * (posx-movetox) + moves
			if posx < movetox:
				moves = [arrow_right] * (movetox-posx) + moves
			if posy > movetoy:
				moves = [arrow_up] * (posy-movetoy) + moves
			if posy < movetoy:
				moves = [arrow_down] * (movetoy-posy) + moves

			print('curpos:',pos)
			print('moveto:',(movetox,movetoy))
			print('new moves:', moves)

		if is_move_prompt(buf) and len(moves) > 0:
			grid = read_grid(buf)
			grid = read_grid_arr(buf)
			print(grid)

			if last_move == 'f' and fire_pattern not in pattern_hash:
				pattern_hash[fire_pattern] = deduce_pattern(last_grid,grid,pos)
				print('pattern found:', fire_pattern, pattern_hash[fire_pattern])

			last_grid = grid
			move = moves[0]
			last_move = move
			moves = moves[1:]

			if move == arrow_right:
				x,y = pos
				pos = min(x+1,24),y
			if move == arrow_left:
				x,y = pos
				pos = max(x-1,0),y
			if move == arrow_down:
				x,y = pos
				pos = x,min(y+1,24)
			if move == arrow_up:
				x,y = pos
				pos = x,max(y-1,0)
			if move.startswith('s'):
				fire_pattern = move

			print(buf, flush=True)
			print('writing:', move.encode('utf-8'))
			writer.write(move)
			full_outp = ''

	print()
	print(pattern_hash)

if __name__ == '__main__':
	loop = asyncio.get_event_loop()
	coro = telnetlib3.open_connection('18.205.93.120', 1203, shell=shell)
	reader, writer = loop.run_until_complete(coro)
	loop.run_until_complete(writer.protocol.waiter_closed)

```

Running this will (eventually) clear the board and get you the flag `AOTW{d0_the_w0r1d_a_favor_and_d0nt_act_like_a_mach1n3}`

