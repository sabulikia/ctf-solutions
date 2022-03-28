## Reversing

- **Riyadh**
It's a non-stripped Position Independant Executable (PIE)

```
> file Reykjavik

Riyadh: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=393ea266da40acda58b1102e4aa0433cbf87174e, for GNU/Linux 3.2.0, not stripped
```

Run with gdb
```
(gdb) set disassembly-flavor intel
(gdb) break main
Breakpoint 1 at 0x1100
```

Our offset in the above case is `100`.

When the breakpoint hits for the first time:
```
(gdb) run 

Breakpoint 1, 0x0000555555555100 in main ()
```

Looking at the mapped addresses:

```
(gdb) info proc mapping
process 14
Mapped address spaces:

          Start Addr           End Addr       Size     Offset objfile
      0x555555554000     0x555555555000     0x1000        0x0 /home/Riyadh
      0x555555555000     0x555555556000     0x1000     0x1000 /home/Riyadh
      0x555555556000     0x555555557000     0x1000     0x2000 /home/Riyadh
      0x555555557000     0x555555558000     0x1000     0x2000 /home/Riyadh
      0x555555558000     0x555555559000     0x1000     0x3000 /home/Riyadh
```

Which matches what we have seen so far that the offset `0x1100` was mapped to `0x0000555555555100`.


**Now to the challenge**

First, disassmble main.
```
(gdb) disassemble main
```

Now first point of interest is 

```
   0x000055555555513f <+63>:	cmp    r12d,0x1
   0x0000555555555143 <+67>:	je     0x55555555525b <main+347>
   0x0000555555555149 <+73>:	call   0x555555555d20 <_Z18CTFLearnHiddenFlagv>
```

Let's put a breakpoint at `0x000055555555513f` and continue

```
(gdb) break *0x000055555555513f
Breakpoint 2 at 0x55555555513f

(gdb) c
```
Now let's print the value of `r12d`
```
(gdb) p/d $r12d
$3 = 1
```

It looks like the comparison is going to be true and the instructions are going to jump to `0x55555555525b`

Now in order to avoid that, let's set the value of `r12d` to something that is not equl to `0x1` and see what happens.

```
(gdb) set $r12d=0x0
(gdb) p/d $r12d
$4 = 0
```

Now before we continue, let's look at another interesting part of the main disassembly

```
   0x000055555555515a <+90>:	mov    rdi,rbp
   0x000055555555515d <+93>:	mov    rsi,r13
   0x0000555555555160 <+96>:	call   0x5555555550e0 <strcmp@plt>
   0x0000555555555165 <+101>:	test   eax,eax
   0x0000555555555167 <+103>:	je     0x555555555286 <main+390>
```

Let's put a breakpoint for the call to `strcmp`.
```
(gdb) break *0x0000555555555160
Breakpoint 3 at 0x555555555160
(gdb) c
Continuing.

Breakpoint 3, 0x0000555555555160 in main ()
```

When the breakpoint hits, let's take a look at the value of `rdi`

```
(gdb) p/s $rdi
$7 = 93824992248256
(gdb) x/s $rdi
0x5555555581c0 <buffer>:	"CTFlearn{Reversing_Is_Easy}"
```

Now is this the flag? I don't think so, because looking at a few lines down, it looks like if the comparision succeeds (what I mean is if the two values are equal), it's jumping to `0x555555555286`. However, let's try this flag anyway.

```
(gdb) run CTFlearn{Reversing_Is_Easy}
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/Riyadh CTFlearn{Reversing_Is_Easy}

Breakpoint 1, 0x0000555555555100 in main ()
(gdb) c
Continuing.
Welcome to CTFlearn Riyadh Reversing Challenge!
Compile Options: ${CMAKE_CXX_FLAGS} -O0 -fno-stack-protector -mno-sse

Breakpoint 2, 0x000055555555513f in main ()
(gdb) c
Continuing.

Breakpoint 3, 0x0000555555555160 in main ()
(gdb) c
Continuing.
You found the false flag!  It's not that easy dude!
```

\* Sigh \* 

Now let's start again and stop right before hitting the third breakpoint.

The next interesting part of the dissassmbled main is 

```
   0x000055555555516d <+109>:	mov    rdi,r13
   0x0000555555555170 <+112>:	call   0x5555555550c0 <strlen@plt>
   0x0000555555555175 <+117>:	cmp    rax,0x1e
   0x0000555555555179 <+121>:	jne    0x555555555243 <main+323>
```
It looks like it's comparing the length of a string to the value `0x1e`.

Let's put a breakpoint at `0x0000555555555175`.

```
(gdb) break *0x0000555555555175 
Breakpoint 4 at 0x555555555175
(gdb) c
```

Now when the breakpoint hits, let's look at the string that was copied:
```
(gdb) x/s $r13
0x7fffffffe95b:	""
```
It's and empty string. It looks like `r13` is holding the command line argument provided. 

This can be verified:
```
(gdb) disassemble main
Dump of assembler code for function main:
=> 0x0000555555555100 <+0>:	endbr64 
   0x0000555555555104 <+4>:	push   r13
   0x0000555555555106 <+6>:	xor    eax,eax
   0x0000555555555108 <+8>:	mov    ecx,0x20
   0x000055555555510d <+13>:	mov    r13,rsi
   .
   .
   .
   0x000055555555514e <+78>:	mov    r13,QWORD PTR [r13+0x8]
```
On the last line, `r13` was moved 8 bytes down which accounts for characters in `./Riyadh` so it only holds the provided value for the flag, in our case an empty string.

Now let's get the decimal value of `0x1e`, and value of `rax`
```
(gdb) p/d 0x1e
$1 = 30
(gdb) p/d $rax
$2 = 0
```

So it looks like, the length of the flag has to be 30. 
In order to continue, let's set the value of `rax` to 30.

```
set $rax=0x1e
```

Before we move on, let's take a look at another interesting part of the disassembly:

```
0x00005555555551a8 <+168>:	cmp    BYTE PTR [rbp+rax*1+0x0],sil
0x00005555555551ad <+173>:	setne  dl
0x00005555555551b0 <+176>:	add    rax,0x1
0x00005555555551b4 <+180>:	add    r12d,edx
0x00005555555551b7 <+183>:	cmp    rax,0x1e
0x00005555555551bb <+187>:	jne    0x5555555551a0 <main+160>
```

This seems like a loop that iterates 30 times.
Let's put a breakpoint at `0x00005555555551b7`


