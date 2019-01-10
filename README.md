# OverTheWire Advent Bonanza 2018 Writeup

Enclosed is my writeup for the 2018 OTW Advent CTF (https://advent2018.overthewire.org). The challenges were tough, but a lot of fun. It seemed like the organizers created each challenge so that it had at least two pieces that needed to be solved before getting the flag. I liked this approach although sometimes it was frustrating when I'd solved some of the challenge, but not all of it. I worked on the CTF solo (team name IoN) and solved a good amount of problems but got stuck on a couple others. At some point I gave up trying to solve every problem and place highly and ended up just working on problems I found the most fun. The two high points of the competition for me were solving "Cryptocow" and "Elvish Art" second.

## Scoring

The final competition results can be found [here](https://advent2018.overthewire.org/dashboard/scoreboard/).

The competition is now archived at the OverTheWire warzone. Details are at [http://overthewire.org/warzone/advent2018](http://overthewire.org/warzone/advent2018).

## Problems

1. [Day0 Challenge - reversing (300)](./day0.md) - __SOLVED__: x86 disassembly - RC4 and bit operations to recover the flag
2. Sanity Check - misc (1): Giveaway in the IRC channel: `AOTW{HoHoHo}`
3. [easteregg1 - web (50)](./easteregg1.md) - __SOLVED__: Follow `/robots.txt`
4. Day 1 [vault1 - misc (300)](./day1.md) - __SOLVED__: Programming problem based on a info leak
5. Day 2 boxy - reversing (200) - __SOLVED__: 
6. Day 3 playagame - net (150) - __SOLVED__: 
7. Day 4 [grille_me - crypto/net (150)](./day4.md) - __SOLVED__: Three consecutive ciphers based off of character patterns
8. Day 5 [Santas PWNSHOP - Pwn (300)](./day5.md) - __SOLVED__: Reverse a binary, find a backdoor, and ROP with a return to libc
9. Day 6 [udpsanta - crypto/pwn/reversing (250)](./day6.md) - __SOLVED__: Protobuf parsing and CBC padding oracle
10. [easteregg2 - web (50)](./easteregg2.md) - __SOLVED*__: base64 encoded flag in an included script
11. Day 7 [Homebrew - crypto/web (200)](./day7.md) - __SOLVED__: self-describing REST API and breaking SRP with a zero key
12. Day 8 Xmastree - reversing (200) - __SOLVED__: 
13. Day 9 [Tiny - misc/pwn (200)](./day9.md) - __SOLVED__: Custom encoding and shellcode
14. [tiny_easteregg - misc (50)](./tiny_easteregg.md) - __SOLVED__: Custom decoding from "tiny" challenge
15. Day 10 [Jackinthebox - reversing (300)](./day10.md) - __SOLVED__: Custom instruction set for reversing followed by exhaustive search for flag
16. Day 11 [Lostpresent - pwn/misc (200)](./day11.md) - __SOLVED__: Sudo specific command gives privileged read
17. Day 12 [Overthecounter - Crypto/web (200)](./day12.md) - __SOLVED__: Fixed key XOR in various locations
18. Day 13 [Honor the Gods - fun (200)](https://github.com/OverTheWireOrg/advent2018-honorthegods): fun challenge, see the link
19. Day 13 [Honor The Gods BONUS - fun (150)](https://github.com/OverTheWireOrg/advent2018-honorthegods): fun challenge (bonus), see the link
20. Day 14 [Naughty or nice - Pwn/crypto (300)](./day14.md) - __SOLVED*__: Truncated string and RSA modulus factoring
21. Day 15 [Santa's little recorders - Net/pwn (200)](./day15.md) - __SOLVED__: Blind command injection in a JPG description field
22. Day 16 [Cryptocow - Web/crypto (200)](./day16.md) - __SOLVED__: Bypassing string encoding using a hash extension attack for command injection
23. Day 17 [Naughtykit - Pwn/misc (100)](./day17.md) - __SOLVED*__: CVE-2018-19788, privesc via systemctl with high UID
24. Day 18 [Claustrophobic - Pwn/misc (250)](./day18.md) - __SOLVED__: Writing and executing arbitrary files in a restricted bash shell
25. Day 19 Pwmanager - Web (200) - __SOLVED*__: 
26. [easteregg3 - web (50)](./easteregg3.md) - __SOLVED*__: Metadata in a trophy image
27. Day 20 [The grinch - Pwn (350)](./day20.md) - __SOLVED*__: Format string vuln, stack pivot, and ROP for a shell
28. Day 21 [Fashionista - Fun (200)](https://github.com/OverTheWireOrg/advent2018-fashionista): fun challenge part 1, see the link
29. Day 21 [Our Hearts - fun (200)](https://github.com/OverTheWireOrg/advent2018-fashionista): fun challenge part 2, see the link; [my donation](https://twitter.com/jwnovak/status/1077224297553412098)
30. Day 22 [Elvish art - Pwn (300)](./day22.md) - __SOLVED__: Craft shellcode with restricted byte values
31. Day 23 Nightmare before christmas - Pwn (350) - __NOT SOLVED__: 
32. Day 24 [Santa's vault 2 - Misc (200)](./day24.md) - __SOLVED__: Interacting with a service and dynamic programming
33. Day 25 Endgame - Pwn/net (300) - __NOT SOLVED__: 
34. lostpresent2 - pwn/misc (200) - __NOT SOLVED__: 
35. Snow Hammer - pwn (400) - __NOT SOLVED__:

__SOLVED*__ were solved, but only after some hints when the competition was over

