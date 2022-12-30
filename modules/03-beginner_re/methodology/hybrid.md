# Dynamic Analysis

Dynamic analysis is simply anything that runs the executable, but in our methodology, we will define it as a more targeted attempt at understanding how the file executes, rather than trying to get context like in Triage.

We used `ltrace` and `strace` in our triage because they are so easy to use, but they definitely fall into Dynamic Analysis.  

(As a fun side note, `ldd` can, in very specific cases, run the binary it is checking. This can be dangerous as many people assume that `ldd` is a static tool. Always keep that in mind when running `ldd`, and take it as a lesson to know your tools)

The primary Dynamic Analysis tool we will be using is GDB and you will learn to love it.

# Static Analysis

The hand in hand counterpart to Dynamic, static analysis is looking at various representations of the executable and deriving understanding from it. 

The primary static tool we will be using is Ghidra, but occasionally also tools like `objdump`. 

# Hybrid Analysis

"What's better, static or dynamic analysis?" 

Obviously it's a mix of the two, having GDB open in one tab with Ghidra open in another, and moving back and forth between the two, taking detailed notes, labeling functions, and figuring out a holistic understanding of how the program works. Your methodology will be your own and eventually you will find what works for you for specific RE types. 
