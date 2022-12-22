
First, let's work through some basic methodology: 

1. Watch [this video from LiveOverflow](https://www.youtube.com/watch?v=VroEiMOJPm8) to give you an idea of what goes into RE. 
   
In order to effectively reverse any executable, you need to scope your approach with the end goal in mind. 

If you are looking at a potential piece of malware, the goals of RE can include:
1. Determine if malware "smells" malicious enough to forward for more in depth analysis
2. Verify executable is truly malicious
3. Identify specific behavior executable runs
4. Figure out communications scheme and decode historical comms 
5. Write anti-virus signatures to detect similar malware in the future. 
6. Write a blog post about cool things the malware does for content marketing

If you are trying to reverse a piece of commercial software goals can include:
1. Identifying how license generation works
2. Bypassing license verification
3. Determine how product works so you can copy functionality
4. Hook specific functions to modify how it works 
5. Identify if there are any security bugs
6. Write 1337 0-day for Remote Code Execution 

For both of the examples given, all of the desired outcomes have a different methodology required to solve the problem, and using the wrong methodology will waste a signifcant amount of time. For example, if the only goal is to identify if something is potentially malicious and should be blocked, once a reverser identifies any sort of covert communications with a remote server, that executable should be flagged and the reverser should move on. Any further work is wasted time, the goal has been reached. If the malware is selected for a more targeted RE later, there needs to be a specific end goal in mind. Without that, reversers can find themselves tearing apart a binary and labeling all the functions for no real reason than that they enjoy it.

While the methodologies might change, in the vast majority of cases, the beginning steps will look the same. This first step in the methodology is often known as "triage". Triage is primarily using CLI tools to get the quickest understanding of an unknown binary. Dynamic analysis is any attempt at figuring out what a binary is doing by executing it and seeing what happens. Static analysis is any attempt to figure out what a binary does by looking at the code, whether it is in a compiled or decompiled form, and  

1. Check out this [lesson on my Triage Methodology](methodology/triage.md): 
2. Then read the [Dynamic and Static Analysis lesson](methodology/hybrid.md)

A good reverse engineering approach uses a blend of all these tools and approaches to come to the required answer. There is no one-size fits all approach, but what we discussed here is a repeatable process that you should use as a base to figure out what works for you. 

Enough talking, let's start with some RE! 

The magic of Nightmare is getting reps and sets in, and that is what you are about to do. Follow these writeups and build that muscle memory.


1. First, we are going to start with a fun CTF challenge that will take us on a whirlwind tour of x86: [Challenge 1](csaw18_x86tour_pt1/)

2. Many RE challenges fall into the category of CrackMe's. I love this [tutorial](https://nora.codes/tutorial/an-intro-to-x86_64-reverse-engineering/) from Nora Codes. [Crack Me Collection](https://github.com/NoraCodes/crackmes/). Work through them and you will know what is needed to solve 50% of CrackMes you find in most competitions. When you get bored, move on, or stop at the 4th challenge. Can't have you spending too much time in one place. 

As a note, NoraCodes uses Radare for the examples, you should be using Ghidra. There is no major functional difference between the way the two work, but you should use Ghidra. (I should do Ghidra writeups of everything but... haven't had the time yet.)

3. How about another CTF challenge? [Challenge 2](pico18_strings/)

4. Helithumper! [Challenge 3](helithumper_re/)

5. Do you beleaf in life after love? [Challenge 4](csaw19_beleaf/)

This is a good array of challenges and you should have some decent understanding of what these types of challenges look like. 

# [NEXT: Chapter 5: Basic Overflows](https://github.com/hoppersroppers/nightmare/blob/master/modules/04-Overflows/readme.md)
