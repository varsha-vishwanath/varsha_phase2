# 1. GDB baby step 1

Can you figure out what is in the eax register at the end of the main function? Put your answer in the picoCTF flag format: `picoCTF{n}` where n is the contents of the eax register in the decimal number base. If the answer was 0x11 your flag would be `picoCTF{17}`.
Disassemble this.

## Solution:
- The 1st hint mentioned that gdb debugger was a good idea to use, so I looked into it and learned that GDB (of GNU debugger) basically allows you to xray a program while it runs in order to see exactly what was happening when it crashed.
- My first instinct was to use an online GDB tool, but the first couple of ones I saw required me to have the code on hand, and I didn't have that with this file.
- Instead I decided to try using it on my ubuntu wsl. So after looking up how to do that, I first loaded the file with gdb and dissassembled the main function as that's what the description said:
```
vasha@Varsha:/mnt/c/users/varsha/downloads$ gdb ./debugger0_a
GNU gdb (Ubuntu 15.0.50.20240403-0ubuntu1) 15.0.50.20240403-git
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./debugger0_a...

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
(No debugging symbols found in ./debugger0_a)
(gdb) disassemble /r main
Dump of assembler code for function main:
   0x0000000000001129 <+0>:     f3 0f 1e fa             endbr64
   0x000000000000112d <+4>:     55                      push   %rbp
   0x000000000000112e <+5>:     48 89 e5                mov    %rsp,%rbp
   0x0000000000001131 <+8>:     89 7d fc                mov    %edi,-0x4(%rbp)
   0x0000000000001134 <+11>:    48 89 75 f0             mov    %rsi,-0x10(%rbp)
   0x0000000000001138 <+15>:    b8 42 63 08 00          mov    $0x86342,%eax
   0x000000000000113d <+20>:    5d                      pop    %rbp
   0x000000000000113e <+21>:    c3                      ret
End of assembler dump.
(gdb)
```
- Then, we can notice that in the main, there's a function `mov` that loads some value into the `eax` register.
- Now, after learning that `$0x86342` was the syntax of a hexadecimal, I simply converted it to get the decimal value needed for the flag:
<img width="893" height="634" alt="image" src="https://github.com/user-attachments/assets/747cb35e-c7d5-454a-932f-98b59d865ade" />

## Flag:

`picoCTF{549698}`

## Concepts learnt:

- Include the new topics you've come across and explain them in brief
- 

## Notes:

- I may have been able to find a website that would've help me dissassemble the file more easily, but i think the method I ended up using helped me become more familiar with using the wsl anyways.

## Resources:

- https://stackoverflow.com/questions/2670639/why-are-hexadecimal-numbers-prefixed-with-0x
- https://www.onlinegdb.com/
- https://sourceware.org/gdb/
- https://cgi.cse.unsw.edu.au/~learn/debugging/modules/gdb_setup/
- https://visualgdb.com/gdbreference/commands/
- https://www.rapidtables.com/convert/number/hex-to-decimal.html?x=86342


# 2. Challenge name

> Description

.
.
.
