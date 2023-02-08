# Unit 2: Reverse Engineering Binary Programs
`From Aviv's 485H`
# Understanding a Binary Program

## A binary program ... what is it?

In the last unit we looked a lot at writing programs in c, compiling
them into binaries, and then running them. In this unit, we peel back
the covers further and look right at the binary files themselves. We
will examine both what exactly is a binary, how is it formatted, and how
do we parse or dissemble the contents within?

## `objdump` and `readelf` basics

For this entire class, we will pick apart a simple helloworld program:

``` c
/*helloworld.c*/
#include <stdio.h>

void printhello(){

  char hello[15]="Hello, World!\n";
  char * p;

  for(p = hello; *p; p++){

    putchar(*p);

  }

}

int main(){

  printhello();

  return 0;
}
}
```

Let's compile the program to create a binary (but with a bunch of
options to make it easier for us):

``` example
$ gcc -fno-stack-protector -z execstack -Wno-format-security -O0 -g    helloworld.c   -o helloworld
```

Now if we use the `file` command we can see what kind of file the binary
is.

``` example
$ file helloworld
helloworld: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2653c851c192779ec91599dc79d28f98bdf66bb4, not stripped
```

We see that it is actually an `ELF` file, which stands for Executable
and Linkable Format. We will work exclusively with binaries in ELF.

### ELF Files and ELF Headers

All ELF files have a header describing the different sections and
general information. We can read the header information for our
`helloworld` program using the `readelf`

``` example
$ readelf -h helloworld
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Intel 80386
  Version:                           0x1
  Entry point address:               0x8048310
  Start of program headers:          52 (bytes into file)
  Start of section headers:          7028 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         9
  Size of section headers:           40 (bytes)
  Number of section headers:         36
  Section header string table index: 33
```

Most of this information isn't too useful, but let me point out some key
things.

1.  There is a magic number! The magic number (like in all file formats) is used to say, hey this
    is ELF and what version
2.  The class is ELF32, so it's 32 bit
3.  The machine is Intel 80386, or x386 is to be expected (this is x86)
4.  The entry point for the file is address 0x8048310, essentially what
    is the first instruction in the \_<sub>start</sub> section function
    which calls main.

Everything else is not super useful for our purposes. Another nice thing
we can do with `readelf` is we can look at all the sections, which is
regions of the binary for different purposes.

``` example
$ readelf -S helloworld
There are 35 section headers, starting at offset 0x2070:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        08048154 000154 000013 00   A  0   0  1
  [ 2] .note.ABI-tag     NOTE            08048168 000168 000020 00   A  0   0  4
  [ 3] .note.gnu.build-i NOTE            08048188 000188 000024 00   A  0   0  4
  [ 4] .gnu.hash         GNU_HASH        080481ac 0001ac 000020 04   A  5   0  4
  [ 5] .dynsym           DYNSYM          080481cc 0001cc 000050 10   A  6   1  4
  [ 6] .dynstr           STRTAB          0804821c 00021c 00004d 00   A  0   0  1
  [ 7] .gnu.version      VERSYM          0804826a 00026a 00000a 02   A  5   0  2
  [ 8] .gnu.version_r    VERNEED         08048274 000274 000020 00   A  6   1  4
  [ 9] .rel.dyn          REL             08048294 000294 000008 08   A  5   0  4
  [10] .rel.plt          REL             0804829c 00029c 000010 08  AI  5  23  4
  [11] .init             PROGBITS        080482ac 0002ac 000023 00  AX  0   0  4
  [12] .plt              PROGBITS        080482d0 0002d0 000030 04  AX  0   0 16
  [13] .plt.got          PROGBITS        08048300 000300 000008 08  AX  0   0  8
  [14] .text             PROGBITS        08048310 000310 000212 00  AX  0   0 16
  [15] .fini             PROGBITS        08048524 000524 000014 00  AX  0   0  4
  [16] .rodata           PROGBITS        08048538 000538 000008 00   A  0   0  4
  [17] .eh_frame_hdr     PROGBITS        08048540 000540 00004c 00   A  0   0  4
  [18] .eh_frame         PROGBITS        0804858c 00058c 00012c 00   A  0   0  4
  [19] .init_array       INIT_ARRAY      08049f0c 000f0c 000004 04  WA  0   0  4
  [20] .fini_array       FINI_ARRAY      08049f10 000f10 000004 04  WA  0   0  4
  [21] .dynamic          DYNAMIC         08049f14 000f14 0000e8 08  WA  6   0  4
  [22] .got              PROGBITS        08049ffc 000ffc 000004 04  WA  0   0  4
  [23] .got.plt          PROGBITS        0804a000 001000 000014 04  WA  0   0  4
  [24] .data             PROGBITS        0804a014 001014 000008 00  WA  0   0  4
  [25] .bss              NOBITS          0804a01c 00101c 000004 00  WA  0   0  1
  [26] .comment          PROGBITS        00000000 00101c 000024 01  MS  0   0  1
  [27] .debug_aranges    PROGBITS        00000000 001040 000020 00      0   0  1
  [28] .debug_info       PROGBITS        00000000 001060 000389 00      0   0  1
  [29] .debug_abbrev     PROGBITS        00000000 0013e9 000130 00      0   0  1
  [30] .debug_line       PROGBITS        00000000 001519 0000dc 00      0   0  1
  [31] .debug_str        PROGBITS        00000000 0015f5 0002bd 01  MS  0   0  1
  [32] .symtab           SYMTAB          00000000 0018b4 000470 10     33  49  4
  [33] .strtab           STRTAB          00000000 001d24 000206 00      0   0  1
  [34] .shstrtab         STRTAB          00000000 001f2a 000145 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)
```

Again, a bunch of this information isn't too useful for us, but might be
later. Some key things to look at:

1.  The `.bss` section is listed, this is the same as bss in the program
    memory layout
2.  There is also .data section, same as the program memory layout
3.  Finally, there is a .text section, same as before. And notice that
    it is at address 0x08048370 which is the same address in the header
    to the start of the program instructions.

### Getting at the assembly with `objdump`

Now that we have some idea of the how the file is formatted, it would be
nice to get down into the details of the machine instructions
themselves. For this, we'll use `objdump` or "object dump". Simply, we
can call it on the binary executable like so:

``` example
$ objdump -d helloworld   

helloworld:     file format elf32-i386


Disassembly of section .init:

080482ac <_init>:
 80482ac:	53                   	push   %ebx
 80482ad:	83 ec 08             	sub    $0x8,%esp
 80482b0:	e8 ab 00 00 00       	call   8048360 <__x86.get_pc_thunk.bx>
 80482b5:	81 c3 4b 1d 00 00    	add    $0x1d4b,%ebx
 80482bb:	8b 83 fc ff ff ff    	mov    -0x4(%ebx),%eax
 80482c1:	85 c0                	test   %eax,%eax
 80482c3:	74 05                	je     80482ca <_init+0x1e>
 80482c5:	e8 36 00 00 00       	call   8048300 <__gmon_start__@plt>
 80482ca:	83 c4 08             	add    $0x8,%esp
 80482cd:	5b                   	pop    %ebx
 80482ce:	c3                   	ret    

    ...
```

It's going to dump a lot of stuff, but lets look more carefully down,
we'll see one header that looks familiar `main()` and `printhello()`:

``` example
08048426 <printhello>:
 8048426:	55                   	push   %ebp
 8048427:	89 e5                	mov    %esp,%ebp
 8048429:	53                   	push   %ebx
 804842a:	83 ec 24             	sub    $0x24,%esp
 804842d:	e8 2e ff ff ff       	call   8048360 <__x86.get_pc_thunk.bx>
 8048432:	81 c3 ce 1b 00 00    	add    $0x1bce,%ebx
 8048438:	c7 45 e5 48 65 6c 6c 	movl   $0x6c6c6548,-0x1b(%ebp)
 804843f:	c7 45 e9 6f 2c 20 57 	movl   $0x57202c6f,-0x17(%ebp)
 8048446:	c7 45 ed 6f 72 6c 64 	movl   $0x646c726f,-0x13(%ebp)
 804844d:	66 c7 45 f1 21 0a    	movw   $0xa21,-0xf(%ebp)
 8048453:	c6 45 f3 00          	movb   $0x0,-0xd(%ebp)
 8048457:	8d 45 e5             	lea    -0x1b(%ebp),%eax
 804845a:	89 45 f4             	mov    %eax,-0xc(%ebp)
 804845d:	eb 19                	jmp    8048478 <printhello+0x52>
 804845f:	8b 45 f4             	mov    -0xc(%ebp),%eax
 8048462:	0f b6 00             	movzbl (%eax),%eax
 8048465:	0f be c0             	movsbl %al,%eax
 8048468:	83 ec 0c             	sub    $0xc,%esp
 804846b:	50                   	push   %eax
 804846c:	e8 7f fe ff ff       	call   80482f0 <putchar@plt>
 8048471:	83 c4 10             	add    $0x10,%esp
 8048474:	83 45 f4 01          	addl   $0x1,-0xc(%ebp)
 8048478:	8b 45 f4             	mov    -0xc(%ebp),%eax
 804847b:	0f b6 00             	movzbl (%eax),%eax
 804847e:	84 c0                	test   %al,%al
 8048480:	75 dd                	jne    804845f <printhello+0x39>
 8048482:	90                   	nop
 8048483:	8b 5d fc             	mov    -0x4(%ebp),%ebx
 8048486:	c9                   	leave  
 8048487:	c3                   	ret    

08048488 <main>:
 8048488:	8d 4c 24 04          	lea    0x4(%esp),%ecx
 804848c:	83 e4 f0             	and    $0xfffffff0,%esp
 804848f:	ff 71 fc             	pushl  -0x4(%ecx)
 8048492:	55                   	push   %ebp
 8048493:	89 e5                	mov    %esp,%ebp
 8048495:	51                   	push   %ecx
 8048496:	83 ec 04             	sub    $0x4,%esp
 8048499:	e8 18 00 00 00       	call   80484b6 <__x86.get_pc_thunk.ax>
 804849e:	05 62 1b 00 00       	add    $0x1b62,%eax
 80484a3:	e8 7e ff ff ff       	call   8048426 <printhello>
 80484a8:	b8 00 00 00 00       	mov    $0x0,%eax
 80484ad:	83 c4 04             	add    $0x4,%esp
 80484b0:	59                   	pop    %ecx
 80484b1:	5d                   	pop    %ebp
 80484b2:	8d 61 fc             	lea    -0x4(%ecx),%esp
 80484b5:	c3                   	ret    
```

This is the assembly for the main function. Looking across, from left to
right, the furthest left is the address this instruction is loaded
into, then the actually bytes of the instruction, and then finally the
name of the details of the instruction.

The first thing you might notice about the instruction itself is that it
is really, really hard to read. That's because it is AT&T syntax, which,
simply, sucks! We will use an alternative format called Intel syntax,
which is much, much nicer. For that, we need to pass an argument to
`objdump`:

``` example
user@si485H-base:demo$ objdump -M intel -d helloworld 

(... snip ...)

08048426 <printhello>:
 8048426:	55                   	push   ebp
 8048427:	89 e5                	mov    ebp,esp
 8048429:	53                   	push   ebx
 804842a:	83 ec 24             	sub    esp,0x24
 804842d:	e8 2e ff ff ff       	call   8048360 <__x86.get_pc_thunk.bx>
 8048432:	81 c3 ce 1b 00 00    	add    ebx,0x1bce
 8048438:	c7 45 e5 48 65 6c 6c 	mov    DWORD PTR [ebp-0x1b],0x6c6c6548
 804843f:	c7 45 e9 6f 2c 20 57 	mov    DWORD PTR [ebp-0x17],0x57202c6f
 8048446:	c7 45 ed 6f 72 6c 64 	mov    DWORD PTR [ebp-0x13],0x646c726f
 804844d:	66 c7 45 f1 21 0a    	mov    WORD PTR [ebp-0xf],0xa21
 8048453:	c6 45 f3 00          	mov    BYTE PTR [ebp-0xd],0x0
 8048457:	8d 45 e5             	lea    eax,[ebp-0x1b]
 804845a:	89 45 f4             	mov    DWORD PTR [ebp-0xc],eax
 804845d:	eb 19                	jmp    8048478 <printhello+0x52>
 804845f:	8b 45 f4             	mov    eax,DWORD PTR [ebp-0xc]
 8048462:	0f b6 00             	movzx  eax,BYTE PTR [eax]
 8048465:	0f be c0             	movsx  eax,al
 8048468:	83 ec 0c             	sub    esp,0xc
 804846b:	50                   	push   eax
 804846c:	e8 7f fe ff ff       	call   80482f0 <putchar@plt>
 8048471:	83 c4 10             	add    esp,0x10
 8048474:	83 45 f4 01          	add    DWORD PTR [ebp-0xc],0x1
 8048478:	8b 45 f4             	mov    eax,DWORD PTR [ebp-0xc]
 804847b:	0f b6 00             	movzx  eax,BYTE PTR [eax]
 804847e:	84 c0                	test   al,al
 8048480:	75 dd                	jne    804845f <printhello+0x39>
 8048482:	90                   	nop
 8048483:	8b 5d fc             	mov    ebx,DWORD PTR [ebp-0x4]
 8048486:	c9                   	leave  
 8048487:	c3                   	ret    

08048488 <main>:
 8048488:	8d 4c 24 04          	lea    ecx,[esp+0x4]
 804848c:	83 e4 f0             	and    esp,0xfffffff0
 804848f:	ff 71 fc             	push   DWORD PTR [ecx-0x4]
 8048492:	55                   	push   ebp
 8048493:	89 e5                	mov    ebp,esp
 8048495:	51                   	push   ecx
 8048496:	83 ec 04             	sub    esp,0x4
 8048499:	e8 18 00 00 00       	call   80484b6 <__x86.get_pc_thunk.ax>
 804849e:	05 62 1b 00 00       	add    eax,0x1b62
 80484a3:	e8 7e ff ff ff       	call   8048426 <printhello>
 80484a8:	b8 00 00 00 00       	mov    eax,0x0
 80484ad:	83 c4 04             	add    esp,0x4
 80484b0:	59                   	pop    ecx
 80484b1:	5d                   	pop    ebp
 80484b2:	8d 61 fc             	lea    esp,[ecx-0x4]
 80484b5:	c3                   	ret    
 
 (... snip ...)
```

### Disassembling with `gdb`

Another way to get the disassembly code is using `gdb`, the gnu debugger,
which also does a tone of other tasks which we will look at later. To
start, run the program under the debugger:

``` example
$ gdb helloworld 
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from helloworld...done.
(gdb) ds printhello
Dump of assembler code for function printhello:
   0x08048426 <+0>:	push   ebp
   0x08048427 <+1>:	mov    ebp,esp
   0x08048429 <+3>:	push   ebx
   0x0804842a <+4>:	sub    esp,0x24
   0x0804842d <+7>:	call   0x8048360 <__x86.get_pc_thunk.bx>
   0x08048432 <+12>:	add    ebx,0x1bce
   0x08048438 <+18>:	mov    DWORD PTR [ebp-0x1b],0x6c6c6548
   0x0804843f <+25>:	mov    DWORD PTR [ebp-0x17],0x57202c6f
   0x08048446 <+32>:	mov    DWORD PTR [ebp-0x13],0x646c726f
   0x0804844d <+39>:	mov    WORD PTR [ebp-0xf],0xa21
   0x08048453 <+45>:	mov    BYTE PTR [ebp-0xd],0x0
   0x08048457 <+49>:	lea    eax,[ebp-0x1b]
   0x0804845a <+52>:	mov    DWORD PTR [ebp-0xc],eax
   0x0804845d <+55>:	jmp    0x8048478 <printhello+82>
   0x0804845f <+57>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048462 <+60>:	movzx  eax,BYTE PTR [eax]
   0x08048465 <+63>:	movsx  eax,al
   0x08048468 <+66>:	sub    esp,0xc
   0x0804846b <+69>:	push   eax
   0x0804846c <+70>:	call   0x80482f0 <putchar@plt>
   0x08048471 <+75>:	add    esp,0x10
   0x08048474 <+78>:	add    DWORD PTR [ebp-0xc],0x1
   0x08048478 <+82>:	mov    eax,DWORD PTR [ebp-0xc]
   0x0804847b <+85>:	movzx  eax,BYTE PTR [eax]
   0x0804847e <+88>:	test   al,al
   0x08048480 <+90>:	jne    0x804845f <printhello+57>
   0x08048482 <+92>:	nop
   0x08048483 <+93>:	mov    ebx,DWORD PTR [ebp-0x4]
   0x08048486 <+96>:	leave  
   0x08048487 <+97>:	ret    
End of assembler dump.
(gdb) ds main
Dump of assembler code for function main:
   0x08048488 <+0>:	lea    ecx,[esp+0x4]
   0x0804848c <+4>:	and    esp,0xfffffff0
   0x0804848f <+7>:	push   DWORD PTR [ecx-0x4]
   0x08048492 <+10>:	push   ebp
   0x08048493 <+11>:	mov    ebp,esp
   0x08048495 <+13>:	push   ecx
   0x08048496 <+14>:	sub    esp,0x4
   0x08048499 <+17>:	call   0x80484b6 <__x86.get_pc_thunk.ax>
   0x0804849e <+22>:	add    eax,0x1b62
   0x080484a3 <+27>:	call   0x8048426 <printhello>
   0x080484a8 <+32>:	mov    eax,0x0
   0x080484ad <+37>:	add    esp,0x4
   0x080484b0 <+40>:	pop    ecx
   0x080484b1 <+41>:	pop    ebp
   0x080484b2 <+42>:	lea    esp,[ecx-0x4]
   0x080484b5 <+45>:	ret    
End of assembler dump.

```

This will print out a disclaimer and leave you in a gdb terminal. Now,
you can type `disassemble <func name>main` to disassemble the main
function. If you set up the alias already as suggested in the resource
page, you can shorten that to `ds` for disassemble.

If your output is in AT&T syntax, then issue the command:

``` example
(gdb) set disassembly-flavor intel
```

To have gdb output in Intel syntax.

I'll mostly work with gdb disassembled output because it's more nicely
formatted, IMHO. Our next task is understanding what the hell is going
on?!?!

## x86 the Processor Register State

### x86 Instruction Set

Let's start with a simple item, what is x86? It is an assembly
instruction set -- a programming language. We can repressed x86 terms of
it's byte (as seen in the objdump output) or in a human readable form
(as seen in the Intel or AT&T) syntax.

You may have worked with an instruction set previously, such as MIPS.
MIPS has the property that it is a RISC instrument set, or a Reduce
Instruction Set Computing, which has the advantage that all instructions
and arguments are always the same size, 32 bits.

x86 is a CISC instruction set, or Complex Instruction Set Computing, and
it has the property that instruction sizes are not consistent. They can
vary between 8 bits and 64 bits and more, depending on the instruction.
You may wonder, why in the world would anything be designed this way?
The answer is market inertia and backwards capability. As Intel chips
dominated the market, more and more binary was x86.

Today, another instruction set has become very relevant: ARM or Acron
Risc Machine. And, as the name indicates, it is a RISC instruction set
and thus is bringing back a bit of sanity to the instruction set world.
ARM is also the architecture of choice on many mobile devices, so it
will be relevant for quite some time.

However, we will not be working with ARM in this class, just x86, and we
will only be using a very small set of the x86 instructions. You can
read more about x86 in the extensive online resources, and when we
encounter an unfamiliar instruction, we will look it up.

### Anatomy of an Instruction

An instruction, in the human readable format, has the following format:

``` example
operation <dst>, <src>
```

The operation name is the kind of operation that will be performed. For
example, it could be an `add` or `mov` or `and`. The `<dst>` is where
the result will be stored, which is typically a register. The `<src>` is
from where the data is read to be operated over which might also include
data referenced in the `<dst>`. The `<src>` is optional, and depends on
the command.

If we take a few operations from our sample code:

``` example
   0x0804841d <+0>: push   ebp
   0x0804841e <+1>: mov    ebp,esp
   0x08048420 <+3>: and    esp,0xfffffff0
```

The first command `push` takes one argument and places that argument on
the stack, adjusting the stack pointer. In this case, it pushes the
value of the base pointer stored in the register `ebp` onto the stack.
The second command, `mov` takes two arguments, and will *move* a value
from one location to another, much like assignment. The second command
moves the value in the stack pointer `esp` and saves it in the base
pointer `ebp`. Finally, the last command is a bitwise operation and
takes two arguments. It will perform a bitwise `and` on the `<dst>` with
the `<src>` and store the result in `<dst>`. In this context, the `and`
command aligns the stack pointer with the lowest 4-bit value. The 4-bit
alignment is due to an old bug in the division unit of the x86
processor, and so you'll see this sequence a lot in assembly.

We will take a closer look at these instructions again in a second, but
before we do that, we need to understand these registers and what they
are used for in more detail.

### Processor Registers

Registers are special storage spaces on the processor that store the
state of the program. Some registers are used for general purpose
storage to store intermediate values, while others are used to keep
track of the execution state, e.g., like what is the next instruction.

Here are the standard registers you will encounter. There are some
others, but we'll explain them when we come across them:

-   `esp`: 32-bit register to store the stack pointer
-   `ebp`: 32-bit register to store the base pointer
-   `eax`: 32-bit general purpose register, sometimes called the
    "accumulator"
-   `ecx`: 32-bit general purpose register
-   `ebx`: 32-bit general purpose register
-   `edx`: 32-bit general purpose register
-   `esi`: 32-bit general purpose registers mostly used for loading and
    storing
-   `edi`: 32-bit general purpose registers mostly used for loading and
    storing

Each of the general purpose registers can be referenced either by their
full 32-bit value or by some subset of that, such as the first 8 bits or
second 8 bits. For example, `eax` refers to the 32-bit general
registers, but `ax` refers to the last 16 bits of the `eax` register and
`al` is the first 8 bits. Depending on the kind of data the register is
storing, we may reference different parts.

[![](imgs/x86-registers.png)](imgs/x86-registers.png)

### The Base Pointer and Stack Pointer

Two registers will be referenced more than any other: the base (`ebp`) and stack (`esp`)
pointer. These registers maintain the memory reference state for the
current execution, with reference to the current function frame. A
function frame is a portion of memory on the stack that stores the
information for a current function's execution, including local data and
return addresses. The base pointer defines the top and bottom of the
function frame.

The structure of a function frame is like so

``` example
           <- 4 bytes ->
          .-------------.    
          |    ...      |    higher address
ebp+0x8 ->| func args   |
ebp+0x4 ->| return addr |       
    ebp ->| saved ebp   |
ebp-0x4 ->|             |
   :      :             :              
   '      '             '
            local args
   .      .             .
   :      :             :
esp+0x4 ->|             |
    esp ->|             |    lower addreses
          '-------------'
```

Moving from higher addresses to lower addresses, the top of the frame
stores the function arguments. These are typically referenced in
positive offsets of the `ebp` register. For example, the first argument is
at `ebp+0x8` moving upwards from there.

The second item in the function frame is the return address at
`ebp+0x4`. The value stored in this memory is where the next instruction
is after the return statement, or what instruction occurs after the call
to this instruction completes. We will spend a LOT OF TIME talking about
this later.

Finally, there is the saved `ebp`, this is the address of the last
base pointer for the calling function. We need to save this value so
that the calling function's stack frame can be restored once this
function completes.

The stack pointer references the bottom of the stack, the lowest address
allocated. Addresses past this point are considered un-allocated.
However, it's pretty easy to allocate more space, we'll just subtract
from the stack pointer.

# Stack Frame Management and Assembly

## Stack Frames

The stack frame is an encapsulation of the local memory state for a
function call. It has information about the local variables of the
function as well as to which function (or specifically which instruction
address) to jump to once this function returns.

A stack frame is managed by two registers: `ebp` and `esp`. The `ebp`
register, or base pointer, is at the top of the stack frame (higher
addresses) and the `esp` register, or stack pointer, is at the bottom of
the stack frame (lower addresses). To make this even more confusing, we
sometimes refer to the `esp` being the "top of the stack" because it represents
the top of the stack in a traditional stack data structure, but it is
really in lower address space.

When we do memory references within a function, we are almost always
describing the address of that memory as a positive or negative offset
of `esp` or `ebp`.

For example, a typical stack frame looks like so:

``` example
           <- 4 bytes ->
          .-------------.    
          |    ...      |    higher address
ebp+0x8 ->| func args   |
ebp+0x4 ->| return addr |       
    ebp ->| saved ebp   |
ebp-0x4 ->|             |
   :      :             :              
   '      '             '
            local vars
   .      .             .
   :      :             :
esp+0x4 ->|             |
    esp ->|             |    lower addreses
          '-------------'
```

The base pointer always references the saved based pointer of the
calling function (more on that soon), and above that (in higher address)
is the return address and the arguments to the function.

## Stack Machines

We often describe x86 processors as stack machines. The reason for this
is that the execution model is built around a stack. As functions are
called, there information is pushed on to the stack, and as functions
return, they are popped off of the stack.

For example, in this code:

``` c

int foo(){
  bar();
  baz();
}



int main(){
  return foo();
}
```

In the stack machine model, the first calling function is pushed on the
stack, namely `main`. Since `main` calls `foo`, we push `foo` onto the
stack next. Now we have something that looks like this:

``` example

   .------.
   | main |
   | foo  |
```

Since foo calls `bar` before it can return, `bar` is pushed on the
stack.

``` example

   .------.
   | main |
   | foo  |
   | bar  |
```

Once `bar` returns, `foo` still can't return since `baz` needs to be
called and is pushed on the stack.

``` example

   .------.
   | main |
   | foo  |
   | baz  |
```

At this point, it is important to note that the entire state of
the functions, `main` and `foo`, have not been forgotten. They
are still there. Once `baz` returns and is popped, `foo` can return and
be popped, and finally, `main` returns and is popped.

This stack-based model of execution is directly applied in the function
stack in memory. Each function is described as a stack frame, and each
stack frame has a link back to the previous stack frame execution
reference (the return address) and alignment information (the saved base
pointer). As functions are called and completed, their stack frames are
pushed and popped off the stack accordingly, but this doesn't happen
automatically. There needs to be explicit instructions to do that, and
that is what we'll look at next.

## Allocating a new stack frame

The first set of instructions in our main function is for managing the
stack frame for the current function. In our code, it looks like this:

``` example
 0x08048426 <+0>:	push   ebp
 0x08048427 <+1>:	mov    ebp,esp
 0x08048429 <+3>:	push   ebx
 0x0804842a <+4>:	sub    esp,0x24
```

Let's first analyze the first four instructions. The push instruction
will push a value onto the stack, and in this case it is the previous
base pointer. Next, the base pointer is set to the stack pointer (`mov`).
Next, subtracting from the stack pointer allocates the rest of the stack 
frame, which is 0x24 bytes long or 36 bytes (don't forget about hex).

The need for these instructions is so that the previous stack frame, the
calling function's stack frame, can be reconstructed. To understand
this, recall the layout of a function's stack frame. The `ebp` register
references the saved value for the base pointer of the calling stack.
Once the function returns, memory is deallocated by adjusting the stack
pointer to the current base pointer, setting the base pointer to the
saved base pointer, and then performing a *leave* by popping off the
saved base pointer and return address, setting the current instruction
pointer to the return address.

Visually the frame construction looks like this:

``` example

    (0) calling         (1) return addr of      (2) push ebp
        function's          calling function   
        stack frame         pushed onto stack

     .-------------.       .-------------.      .-------------.       
     |    ...      |       |    ...      |      |    ...      |  
ebp->|             |  ebp->|             | ebp->|             |
     :             :       :             :      :             :
         calling              calling             calling
     :   stack     :       :   stack     :      :   stack     :
     |   frame     |       |   frame     |      |   frame     |
     |             |       |             |      |             |
esp->|  func args  |       |  func args  |      |  func args  |   
     '-------------'  esp->|  return adr |      |  return adr |
                           '-------------' esp->|  saved ebp  |
                                                '-------------'



       (3) mov ebp,esp         (4) sub esp,0x24

         .-------------.        .-------------.       
         |    ...      |        |    ...      |   
         |             |        |             |  
         :             :        :             :  
            calling                 calling    
         :   stack     :        :   stack     :
         |   frame     |        |   frame     |
         |             |        |             |
         |  func args  |        |  func args  |
         |  retur adr |         |  retur adr  |
esp,ebp->|  saved ebp  |    bp->|  saved ebp  |
         '-------------'        |             |
                                :             :
                                      New 
                                :   stack     :
                           esp->|      Frame  |
                                '-------------'
```

## The `__x86.get_pc_thunk` functions

You may have noticed that in the disassembled x86 for both `main` and
`printhello` is two extra function calls, `__x86.get_pc_thunk.bx` and
`__x86.get_pc_thunk.ax`. These two functions are added by the compiler to load
the address of the *next* instruction into the `bx` and `ax` portion of the
register, respectively. The `add` that follows clears out the lower `bh/bl` and
`ah/al` portion of the registers, so the two work in combination.

Why is this useful? Well, it isn't, at least, not for our programs. This is used
for handling position independent compilation, which is necessary for dynamic
libraries. The ability to load certain program-counter points comes more
naturally in x86_64, but since we are compiling 32-bit on a 64-bit machine
(e.g., the `-m32` and `-no-pie` option to compilation), `gcc` adds in these
functions where needed. They have no real effect on the 32-bit programs we are
analyzing, so you can essentially ignore them.

However, if you're curious how it works, you can use `gdb` to print the
instructions:

``` example

(gdb) x/2i 0x8048360
   0x8048360 <__x86.get_pc_thunk.bx>:	mov    ebx,DWORD PTR [esp]
   0x8048363 <__x86.get_pc_thunk.bx+3>:	ret 
```

Since we reached the function via a `call`, the return address is pushed onto
the stack. `esp` is reference that return address. The `mov` instruction, along
with the deference of `DWORD PTR [esp]`, saves the return address into the `ebx`
register. This way of quickly saving an address will come in handy later when 
we start writing shell code.

Finally, the `mov` instruction at the end of the `printhello` instruction is
also connected to this compiler addition. Since the instruction of the start of
the function is loaded into `ebx`, it places the return address of the function
`printhello` stored at `ebp-0x4`. This, again, is to track execution to enable
position independent code. You can mostly ignore these extra instructions, but
be careful to verify first that it is not effecting the normal operations of the
program you are analyzing.

## De-allocating a stack frame

Now that a new stack frame construction is complete, we can jump to the end and
look at what happens once the function returns. How is the stack frame
deallocated and popped off the stack.

Two instructions manage that process.

``` example
 0x08048486 <+96>:	leave  
 0x08048487 <+97>:	ret   
```

First the `leave` instruction will do two things:

1.  `mov esp,ebp` : set the stack pointer to the base pointer, essential
    deallocating the local variables in the function
2.  `pop ebp` : after the move, the top of the stack is the saved base
    pointer, so by popping the value at the top of the stack into `ebp`, we
    are reseting the base pointer to the saved one.

Visually this would look like this:

``` example
                              leave                  leave
                              1. =mov esp,ebp=       2. pop ebp
      .-------------.          .-------------.      .-------------.   
      |    ...      |          |     ...     | ebp->|    ...      |
      |             |          |             |      |             |
      :             :          :             :      :             :
        calling                  calling               calling
      :   stack     :          :  stack      :      :  stack      :
      |   frame     |          |  frame      |      |  frame      |
      |             |          |             |      |
      |  func args  |          |  func args  |      |  func args  |
      |  retur val  |          |  return adr | esp->|  return adr |
 ebp->|  saved ebp  | ebp,esp->| saved ebp   |      '-------------'
      |             |          '-------------'
      :             :
            New 
      :   stack     :
 esp->|      Frame  |
      '-------------'
```

At this point the stack is almost back to its state prior to the
function call. The last thing to do is to pop off the return value and
set the instruction pointer (i.e. the current execution point) to the
return value. Conceptually, we can see that `ret` does both those in one
step:

1.  `pop eip` : pop the return value and set it to the instruction
    pointer.


``` example
                             ret                 
                            1. pop eip          
      .-------------.      .-------------.     
ebp-> |    ...      | ebp->|      ...    |        
      :             :      :             :         
         calling            calling               
      :  stack      :      : stack       :        
      |  frame      |      | frame       |
      |             |      |             |
      |  func args  | esp->| func args   |
esp-> | return adr  |      '-------------'
      '-------------'
```

# Memory References, Jumps/Loops, and Function Calls

## Referencing, De-Referencing, and Setting Memory

The next set of instructions we will observe initializes the memory of
the stack. Let's switch back to the C-code to see this in C before
we look at it in assembly.

``` c
    char hello[15]="Hello, World!\n";
```

The string "Hello World!\\n" is set on the stack in a 15 byte character
array. In assembly, it looks like this.

``` example
 0x08048438 <+18>:	mov    DWORD PTR [ebp-0x1b],0x6c6c6548
 0x0804843f <+25>:	mov    DWORD PTR [ebp-0x17],0x57202c6f
 0x08048446 <+32>:	mov    DWORD PTR [ebp-0x13],0x646c726f
 0x0804844d <+39>:	mov    WORD PTR [ebp-0xf],0xa21
 0x08048453 <+45>:	mov    BYTE PTR [ebp-0xd],0x0
```

If you squint at the `<src>` of the operators, you'll recognize that
this is ASCII. If you don't believe me, check out the ASCII table. The
DWORD or WORD or BYTE PTR are deference commands.

-   `BYTE PTR[addr]` : byte-pointer : de-reference one byte at the
    address
-   `WORD PTR[addr]` : word-pointer : de-reference the two bytes at the
    address
-   `DWORD PTR[addr]` : double word-pointer : de-reference the four
    bytes at the address

Another way to look at these instructions in C would be like this (don't
program like this, though):

``` c
  char hello[15];
  //                      l l e H  
  * ((int *) (hello)) = 0x6c6c6548;      // set hello[0]->hello[3]
  //                          W   , o
  * ((int *) (hello + 4)) = 0x57202c6f; // set hello[4]->hello[7]
  //                          d l r o      
  * ((int *) (hello + 8)) = 0x646c726f; // set hello[8]->hello[11]
  //                             \n !
  * ((short *) (hello + 12)) = 0x0a21;  // set hello[12]->hello[13]
  //                         \0
  * ((char *) (hello+14)) = 0x00;  // set hello[14]
```

The next two instructions are a bit different:

``` example
 0x08048457 <+49>:	lea    eax,[ebp-0x1b]
 0x0804845a <+52>:	mov    DWORD PTR [ebp-0xc],eax
```

=lea= stands for *load effective address* and is a short cut to do a
bit a math and calculate a pointer offset and store it. If we look at
what's next in the C-program, we see that it is setting up the for-loop.

``` c
   for(p = hello; *p; p++){
```

The first part of the for loop is initializing the pointer `p` to
reference the start of the string `hello`. From the previous code, the
start of the string `hello` is at address offset `ebp-0x1b` and we want to
set that address to the value of `p`. This is a two step process:

1.  The actual address must be computed using addition from `esp` and
    stored. `lea eax,[ebp-0x1b]` will calculate the address and store it
    in `eax`.
2.  The value in `eax` must be stored in the memory reserved for `p`,
    which is at address `ebp-0xc`, the move command accomplishes that.

At this point, everything is set up. And for reference, remember that the
address of `p` is at `ebp-0xc`.

## Loops, Jumps, and Condition Testing

Now, we've reached the meat of the program: the inner loop. We can
follow the execution at this point by following the jumps.

``` example
   0x0804845d <+55>:	jmp    0x8048478 <printhello+82>  # ------------.                          
   0x0804845f <+57>:	mov    eax,DWORD PTR [ebp-0xc]    # <------.    |                          
   0x08048462 <+60>:	movzx  eax,BYTE PTR [eax]         #        |    |                          
   0x08048465 <+63>:	movsx  eax,al                     #        |    |                          
   0x08048468 <+66>:	sub    esp,0xc                    #        |    |                          
   0x0804846b <+69>:	push   eax                        #        |    | //exit condition check   
   0x0804846c <+70>:	call   0x80482f0 <putchar@plt>    #        |    |                          
   0x08048471 <+75>:	add    esp,0x10                   #        |    |                          
   0x08048474 <+78>:	add    DWORD PTR [ebp-0xc],0x1    #        |    |                          
   0x08048478 <+82>:	mov    eax,DWORD PTR [ebp-0xc]    # <------+----'                          
   0x0804847b <+85>:	movzx  eax,BYTE PTR [eax]         #        | //loop body                   
   0x0804847e <+88>:	test   al,al                      #        |                               
   0x08048480 <+90>:	jne    0x804845f <printhello+57>  # -------'                               
```

A `jmp` instruction changes the instruction pointer to the destination
specified. It is not conditional, it is a hard jump. Following that jump 
in the code, we find the following three instructions which are testing 
the exit condition from the loop.

``` example
  0x08048478 <+82>:	mov    eax,DWORD PTR [ebp-0xc]
  0x0804847b <+85>:	movzx  eax,BYTE PTR [eax]
  0x0804847e <+88>:	test   al,al
  0x08048480 <+90>:	jne    0x804845f <printhello+57>
```

Recall that `ebp-0xc` is the address of the pointer `p`, so doing
`DWORD PTR` will move into `eax` the memory reference stored in `p`. At
this point, `eax` stores the value of `p` which references some `char`
byte in the string, so we want to retrieve that `char` byte by
dereferencing `p` further, this time with a `BYTE PTR`.

The problem is, we just placed one byte into a 4-byte register that used
to have other bytes in it. We need to do some cleanup. The cleanup
operation is `movzx` instructions. The `movzx` instruction with the
`BYTE PTR` will *zero extend* the byte into `eax`. This means, that the
remaining 3 bytes of the register will be set to zero. At this point, we
have one byte snuggly stored in the `eax` register that is the `char`
referenced by `p`. We now need to check if that byte is NULL.

That test occurs `test al,al` which compares to registers in a number of
ways. Here we are testing the `al` register which is the lower 8-bits
of `eax`, where we stored the deference of `p`. The results of the test,
greater then, less than, equal, not zero, etc. are stored in a set of
bit flags. The one we care about is the `ZF` flag or the *zero flag*. If
`al` is zero then `ZF` is set to 1 which would be the case when `p`
references the end of the `hello` string.

The `jne` command says to *jump when not equal to zero*. If `al` is zero, 
do not jump, otherwise continue to the address and continue the loop.

## Function Calls

If we investigate the loop body, we find the following instructions:

``` example
   0x0804845f <+57>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048462 <+60>:	movzx  eax,BYTE PTR [eax]
   0x08048465 <+63>:	movsx  eax,al
   0x08048468 <+66>:	sub    esp,0xc
   0x0804846b <+69>:	push   eax
   0x0804846c <+70>:	call   0x80482f0 <putchar@plt>
   0x08048471 <+75>:	add    esp,0x10
```

The first set of instructions, much like the `test` before, is to
deference the pointer `p`.

1.  Load the value of `p`, a memory address, into `eax`
2.  Read the byte referenced at `p` into the lower 8-bits of `eax`
3.  Zero out the remaining bits of `eax` leaving only lower 8-bits

At this point, `eax` stores a value like 0x0000048 (i.e. 'H') where the
lowest byte is the character of interest, and the remaining bytes are 0.

The new instruction is `movsx`, which stands for sign extend. The two
previous operations placed a `char` into the `eax` register, but the
function we are about to call (`putchar()`) expects a signed integer. We
need to cast our one-byte signed value to a four-byte signed value. The
`movsx` operation takes the leading bit of the one-byte value, the sign
parity bit, and extends it to the front of the register where a 4-byte
signed value should be, thus completing the cast.

The next instruction should be familiar, it is a subtraction on the
stack pointer. This allocates space on the stack, providing us a location
to write the argument to the function we are about to call. We allocate
more space than we need in case the function does some stack
manipulation, and then we finally `push` our argument onto the stack.

The next operation is a `call` which will execute the function
`putchar`, conveniently told to us by gdb. Once that function completes,
execution will continue to the point right after the `call`, which is
the instruction `add esp,0x10`. This will deallocate the stack space we
allocate prior to the call.

The final instruction to consider is

``` example
    0x08048474 <+78>:	add    DWORD PTR [ebp-0xc],0x1
```

=p=, and the instructions following test weather `p` now references
zero. And the loop goes on to check the exit condition, which may jump
to do it all over again.
 

 
