Outside of stack based overflows, there are only a few other super common types of challenges. 

We'll start with the most common and important of these, format strings.

# Format Strings

1. Go through [these slides from MBE on format strings](06_lecture.pdf)
2. Read this [lecture on Format Strings from Aviv.](unit_06.md). DO NOT attempt to do the second half on writing with format strings, just read up to "Overwriting the Return Address" and move on. Trust me, it's really complicated, fairly overkill, and doesn't really get seen often.
3. Do your first format string challenge. [Challenge 1](10-fmt_strings/backdoor17_bbpwn/readme.md)
4. [Challenge 2](10-fmt_strings/pico18_echo/readme.md)
5. [Challenge 3](10-fmt_strings/tw16_greeting/readme.md)

# Bad Seed

These are way less common but still pop up all the time. The basic idea is if you know how the randomness works, it's not actually random.

1. [Challenge 4](09-bad_seed/h3_time/readme.md)

Just do one of these, they are are a bit rare, but good to know they exist. If you want another example here are two, but just move on.

* [Challenge 5](09-bad_seed/hsctf19_tuxtalkshow/readme.md)
* [Challenge 6](09-bad_seed/sunshinectf17_prepared/readme.md)

# Partial Overwrite

Partial overwrites are where we get after the low values of an address. They are most commmonly used for ASLR defeat, but are also useful when we only can overwrite part of the return address.

1. [tu17_vulnchat2](15-partial_overwrite/tuctf17_vulnchat2)

Just do one of these, they are are a bit rare, but good to know they exist. If you want another example here are two, but just move on.


* [tamu19_pwn2](15-partial_overwrite/tamu19_pwn2)
* [hacklu15_stackstuff](15-partial_overwrite/hacklu15_stackstuff)


# [NEXT: Chapter 7: ROP](https://github.com/hoppersroppers/nightmare/blob/master/modules/06-ROP/readme.md)
