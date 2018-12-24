# easteregg1 - web (50)

Pretty obvious for a web easter egg - just follow `/robots.txt`:

```
$ curl https://advent2018.overthewire.org/robots.txt
User-agent: Haxx0rs
Disallow: /static/__s3cret.txt
$ curl https://advent2018.overthewire.org/static/__s3cret.txt
Snooping around, are we?

...

AOTW{D0ra_th3_haxxpl0rer}
```
