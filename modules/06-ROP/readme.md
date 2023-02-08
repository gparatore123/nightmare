Time to learn about Return Oriented Programming! 

This is very similar to what you were doing with the callfunction stack overflows, but this time we are going to be stringing sequence of them together. 

# Why ROP? 

Due to the presence of non-executable (NX) stacks, also known as Data Execution Protection (DEP), we find ourselves unable to put instructions onto the stack and execute them. 

Read over this presentation from RPISec to learn more. [DEP Lecture](07_lecture.pdf)

Now read this document without following along to try to get a better mental model of what is happening: [Rop FTW Writeup](https://www.exploit-db.com/docs/english/28479-return-oriented-programming-(rop-ftw).pdf)

Now it's time to actually do some Ropping. 

# How to ROP

Follow Aviv's ROP intro lab to work through your first ROP.

[Aviv's Unit 07 ROP](unit_07.md)


# ROP Chain Statically Compiled

When writing ROP chains, the best thing possible for us is that is the binary is statically compiled. This is because we want as many instructions as possible so we have more potential ROP gadgets. 

For this section, we'll be going through some static binaries which will give us a lot to work with.

1. [DCQuals19 Speedrun](07-bof_static/dcquals19_speedrun1/)
2. [BKP16-SimpleCalc](07-bof_static/bkp16_simplecalc)
3. [DCQuals16-Feedme](07-bof_static/dcquals16_feedme)

# ROP Dynamically Compiled

This time the binaries are dynamic, which means we have less to work with. Not the end of the world, but we'll have to put more work into finding the right gadgets. 

Read this: <https://www.ired.team/offensive-security/code-injection-process-injection/binary-exploitation/return-to-libc-ret2libc>


There is also now much more concern about having the correct libc versions as your target. Because we are only trying to run this on our own systems it is less of a problem, but in the real world, if we are putting in the wrong offsets to items in libc we are going to have a bad time. This is useful if you know the libc version but don't have a copy locally: <https://libc.blukat.me/>

For these writeups, your libc will probably be slightly different, so write the exploit for your local device, don't try to match the writeups' libc.

1. [csaw17-svc](08-bof_dynamic/csawquals17_svc)
2. [fb19-overfloat](08-bof_dynamic/fb19_overfloat)
3. [hs19-storytime](08-bof_dynamic/hs19_storytime)
4. [csaw19-babyboi](08-bof_dynamic/csaw19_babyboi)
5. [utc19-shellme](08-bof_dynamic/utc19_shellme)

# Ret2System

Also the name of a really cool company that teaches exploitation and writes sweet 0days, <https://ret2.io/> ret2system is a more specific versiion of ret2libc that hunts the system() call. 

1. [asis17_marymorton](14-ret_2_system/asis17_marymorton)
2. [hxp18_poorcanary](14-ret_2_system/hxp18_poorCanary)
3. [tu_guestbook](14-ret_2_system/tu_guestbook)




