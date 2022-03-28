## Reversing

- **Reykjavik**

It's a non-stripped Position Independant Executable (PIE)

```
> file Reykjavik

Reykjavik: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=9bc04368dbcefb4491573ac8feea3a32e31ed59f, for GNU/Linux 3.2.0, not stripped
```

Run with gdb

```
gdb Reykjavik

(gdb) break main
Breakpoint 1 at 0x10a0

> run CTFlearn{aaaaaaaa}
```

Last three characters of the address are the offset i.e. in above example `0a0`

When the breakpoint hits for the first time:

```
Breakpoint 1, 0x00005555555550a0 in main ()
```

and doing the following command shows the mapped address spaces:

```
(gdb) info proc mapping
process 1601
Mapped address spaces:

          Start Addr           End Addr       Size     Offset objfile
      0x555555554000     0x555555555000     0x1000        0x0 /home/Reykjavik
      0x555555555000     0x555555556000     0x1000     0x1000 /home/Reykjavik
      0x555555556000     0x555555557000     0x1000     0x2000 /home/Reykjavik
      0x555555557000     0x555555558000     0x1000     0x2000 /home/Reykjavik
      0x555555558000     0x555555559000     0x1000     0x3000 /home/Reykjavik
```

Important information from above:

```

     Start Addr           End Addr       Size     Offset objfile
 0x555555555000     0x555555556000     0x1000     0x1000 /home/Reykjavik
```

Which matches what we have seen so far that the offset `0x10a0` was mapped to `0x00005555555550a0`.

So now for putting breakpoints, let's look at the below example

```
(gdb) set disassembly-flavor intel
(gdb) disassemble main
Dump of assembler code for function main:
   0x00000000000010a0 <+0>:		endbr64
   0x00000000000010a4 <+4>:		push   r13
   0x00000000000010a6 <+6>:		push   r12
   0x00000000000010a8 <+8>:		push   rbp
   0x00000000000010a9 <+9>:		sub    rsp,0x20
   0x00000000000010ad <+13>:	cmp    edi,0x1
   0x00000000000010b0 <+16>:	je     0x11b5 <main+277>
   0x00000000000010b6 <+22>:	mov    rbp,QWORD PTR [rsi+0x8]
```

If I want to put a breakpoint at the below line:

```
0x00000000000010ad <+13>:	cmp    edi,0x1
```

I have to use the following command in gdb:

```
break *0x55555555550ad
```

Also , note that this works because running in gdb means ASLR is disabled by default.

<br>

**Now to the challenge**

First point of interest:

```
   0x00000000000010ad <+13>:	cmp    edi,0x1
   0x00000000000010b0 <+16>:	je     0x11b5 <main+277>
```

My guess is, this is comparing the number of arguments you have provided to send you to usage message if you've provided no command line arguments.

Verify this hypothesis by running the application with no command line argument:

```
(gdb) run

```

The next one is

```
   0x00000000000010e8 <+72>:	mov    rsi,rbp
   0x00000000000010eb <+75>:	mov    rdi,rdx
   0x00000000000010ee <+78>:	repz cmps BYTE PTR ds:[rsi],BYTE PTR es:[rdi]
   0x00000000000010f0 <+80>:	seta   al
   0x00000000000010f3 <+83>:	sbb    al,0x0
   0x00000000000010f5 <+85>:	test   al,al
   0x00000000000010f7 <+87>:	je     0x11c9 <main+297>
```

After running this by inserting a breakpoint at `0ee`, this seems to be a loop. The first time the breakpoint hits, I tried the following:

```
(gdb) x/s $rbp
0x7fffffffe948:	"CTFlearn{aaaaaa}"
(gdb) x/s $rdx
0x5555555560b0:	"CTFlearn{Is_This_A_False_Flag?}"
```

So my guess is this is a false flag and the loop is comparing your provided flag with this value and exiting if they're equal.

Verify this hypothesis by running the application with the supposedly false flag.

```
(gdb) run CTFlearn{Is_This_A_False_Flag?}

```

Next point of interest is :

```
   0x0000000000001168 <+200>:	call   0x1080 <strcmp@plt>
   0x000000000000116d <+205>:	mov    r12d,eax
   0x0000000000001170 <+208>:	test   eax,eax
   0x0000000000001172 <+210>:	jne    0x1197 <main+247>
```

It's calling `strcmp()` and from the lines above it (which are not pasted here) it looks like a bunch of values are being copied from `$rsp` so I left a breakpoint at `168` and after the breakpoint was hit, I tried the following:

```
(gdb) x/s $rsp
0x7fffffffe630:	"CTFlearn{Eye_L0ve_Iceland_}"
```

And voila!

It's usually best to test the flag before celebrating :D, but I was very sure this is the flag for this challenge.

Verify this hypothesis by running the application with the supposedly flag.

```
(gdb) run CTFlearn{Eye_L0ve_Iceland_}

```

Summary of breakpoints:

```
1       breakpoint     keep y   0x00005555555550a0 <main>
	breakpoint already hit 1 time
3       breakpoint     keep y   0x00005555555550ad <main+13>
	breakpoint already hit 1 time
4       breakpoint     keep y   0x00005555555550ee <main+78>
	breakpoint already hit 10 times
5       breakpoint     keep y   0x0000555555555168 <main+200>
```

Decrypt source code:

```
openssl enc -d -aes-256-cbc -pbkdf2 -k CTFlearn{Eye_L0ve_Iceland_} -in sources.zip.enc -out sources.zip
```
