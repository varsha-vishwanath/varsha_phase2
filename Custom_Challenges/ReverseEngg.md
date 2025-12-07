# Reverse Engineering

## JoyDivision

### Solution:



## worthy.knight

### Solution:
After checking the file type, I tried running the binary and saw that it probably returned the flag in return for the correct incantation (10 characters). So I opened `./worthy.knight` with gdb and set a breakpoint at `fgets` (when input enters the program). Then I ran the program and checked the memory at the point input was given. Then I dissassembled `x/40i $pc` and solved for each expected character accordingly which got me the incantation: `NjkSfTYaIi`. Entering that in the binary gave me the flag.

| Byte | How Determined                                                                 | Value | Char |
|------|--------------------------------------------------------------------------------|--------|------|
| 0    | `byte[0] ^ byte[1] = 0x24`, `byte[1] = 'j'` → `'j' ^ 0x24 = 'N'`               | 0x4E  | N    |
| 1    | Direct comparison: must equal `0x6a`                                           | 0x6A  | j    |
| 2    | Direct comparison: must equal `0x6b`                                           | 0x6B  | k    |
| 3    | Direct comparison: must equal `0x53`                                           | 0x53  | S    |
| 4    | MD5(`byte[5] + byte[4]`) must match target hash → brute force → `f`            | 0x66  | f    |
| 5    | MD5(`'T' + 'f'`) matches target hash → `T`                                     | 0x54  | T    |
| 6    | XOR constraint with `byte[7]` → `Y`                                            | 0x59  | Y    |
| 7    | Direct comparison: must equal `0x61`                                           | 0x61  | a    |
| 8    | Direct comparison: must equal `0x49`                                           | 0x49  | I    |
| 9    | XOR constraint → `i`                                                           | 0x69  | i    |

```
vasha@Varsha:/mnt/c/users/varsha/downloads$ gdb ./worthy.knight
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
Reading symbols from ./worthy.knight...

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
Downloading separate debug info for /mnt/c/users/varsha/downloads/worthy.knight
(No debugging symbols found in ./worthy.knight)
(gdb) break fgets
Breakpoint 1 at 0x10c0
(gdb) run
Starting program: /mnt/c/users/varsha/downloads/worthy.knight
Downloading separate debug info for system-supplied DSO at 0x7ffff7fc3000
Downloading separate debug info for /lib/x86_64-linux-gnu/libcrypto.so.3
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Download failed: Invalid argument.  Continuing without source file ./libio/./libio/iofgets.c.
                       (Knight's Adventure)

         O
        <M>            .---.
        /W\           ( -.- )--------.
   ^    \|/            \_o_/         )    ^
  /|\    |     *      ~~~~~~~       /    /|\
  / \   / \  /|\                    /    / \
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Welcome, traveler. A mighty dragon blocks the gate.
Speak the secret incantation (10 runic letters) to continue.

Download failed: Invalid argument.  Continuing without source file ./libio/./libio/iofgets.c.

Breakpoint 1, _IO_fgets (buf=0x7fffffffdd10 "", n=128, fp=0x7ffff7a8f8e0 <_IO_2_1_stdin_>) at ./libio/iofgets.c:32
warning: 32     ./libio/iofgets.c: No such file or directory
(gdb) finish
Run till exit from #0  _IO_fgets (buf=0x7fffffffdd10 "", n=128, fp=0x7ffff7a8f8e0 <_IO_2_1_stdin_>) at ./libio/iofgets.c:32
Enter your incantation: asdfghjklm
0x0000555555555160 in ?? ()
Value returned is $1 = 0x7fffffffdd10 "asdfghjklm\n"
(gdb) x/40i $pc
=> 0x555555555160:      test   %rax,%rax
   0x555555555163:      je     0x5555555552e7
   0x555555555169:      mov    %rbx,%rdi
   0x55555555516c:      lea    0x1717(%rip),%rsi        # 0x55555555688a
   0x555555555173:      call   0x555555555040 <strcspn@plt>
   0x555555555178:      mov    %rbx,%rdi
   0x55555555517b:      movb   $0x0,0x50(%rsp,%rax,1)
   0x555555555180:      call   0x555555555060 <strlen@plt>
   0x555555555185:      cmp    $0xa,%rax
   0x555555555189:      jne    0x555555555234
   0x55555555518f:      call   0x555555555070 <__ctype_b_loc@plt>
   0x555555555194:      lea    0x5a(%rsp),%rdi
   0x555555555199:      mov    (%rax),%rsi
   0x55555555519c:      mov    %rbx,%rax
   0x55555555519f:      movzbl (%rax),%ecx
   0x5555555551a2:      movzbl 0x1(%rax),%edx
   0x5555555551a6:      movzwl (%rsi,%rcx,2),%ecx
   0x5555555551aa:      test   $0x4,%ch
   0x5555555551ad:      je     0x5555555552ca
   0x5555555551b3:      movzwl (%rsi,%rdx,2),%edx
   0x5555555551b7:      test   $0x4,%dh
   0x5555555551ba:      je     0x5555555552ca
   0x5555555551c0:      test   $0x1,%ch
   0x5555555551c3:      je     0x5555555551ce
   0x5555555551c5:      test   $0x1,%dh
   0x5555555551c8:      jne    0x5555555553ec
   0x5555555551ce:      and    $0x2,%ch
   0x5555555551d1:      je     0x5555555551dc
   0x5555555551d3:      and    $0x2,%dh
   0x5555555551d6:      jne    0x5555555553ec
   0x5555555551dc:      add    $0x2,%rax
   0x5555555551e0:      cmp    %rdi,%rax
   0x5555555551e3:      jne    0x55555555519f
   0x5555555551e5:      movzbl 0x51(%rsp),%eax
   0x5555555551ea:      mov    %eax,%edx
   0x5555555551ec:      xor    0x50(%rsp),%edx
--Type <RET> for more, q to quit, c to continue without paging--RET
   0x5555555551f0:      cmp    $0x24,%dl
   0x5555555551f3:      jne    0x555555555296
   0x5555555551f9:      cmp    $0x6a,%al
   0x5555555551fb:      jne    0x5555555552b0
(gdb) quit
A debugging session is active.

        Inferior 1 [process 4103] will be killed.

Quit anyway? (y or n) y
vasha@Varsha:/mnt/c/users/varsha/downloads$ ./worthy.knight
                       (Knight's Adventure)

         O
        <M>            .---.
        /W\           ( -.- )--------.
   ^    \|/            \_o_/         )    ^
  /|\    |     *      ~~~~~~~       /    /|\
  / \   / \  /|\                    /    / \
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Welcome, traveler. A mighty dragon blocks the gate.
Speak the secret incantation (10 runic letters) to continue.

Enter your incantation: NjkSfTYaIi

   The kingdom's gates open, revealing the hidden realm...
                         ( (
                          \ \
                     .--.  ) ) .--.
                    (    )/_/ (    )
                     '--'      '--'
    "Huzzah! Thy incantation is true. Onward, brave knight!"

The final scroll reveals your reward: KCTF{NjkSfTYaIi}

vasha@Varsha:/mnt/c/users/varsha/downloads$
```

### Flag:
**Flag:** `KCTF{NjkSfTYaIi}`



## 

