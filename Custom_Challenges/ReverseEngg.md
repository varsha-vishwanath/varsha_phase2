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



## time

### Solution
I wasn't able to completely solve this challenge but here's what I did and what I've understood:

I used gdb to analyze the binary and realized that it generated a random number every run, compared it with user input, and ran `giveFlag()` if the input matches. I set a breakpoint to stop the run before the comparison happens and then looked into the memory to see the required number. I printed out that number then continued the program to enter it. Turns out I found the secret number, but wasn't able to get the flag as it was not stored in the accessed location. The binary printed `contact a H3 admin for assistance`. The program attempts to open `/home/h3/flag.txt` but I didn't know where it was or how to access it. Insteas, I created an accessible flag file locally and tested it to make sure I was right and ran it. That worked, but it was obviously not the real flag. 

```
vasha@Varsha:/mnt/c/users/varsha/downloads$ gdb ./time
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
Reading symbols from ./time...

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
Downloading separate debug info for /mnt/c/users/varsha/downloads/time
(No debugging symbols found in ./time)
(gdb) disassemble main
Dump of assembler code for function main:
   0x000000000040092b <+0>:     push   %rbp
   0x000000000040092c <+1>:     mov    %rsp,%rbp
   0x000000000040092f <+4>:     sub    $0x20,%rsp
   0x0000000000400933 <+8>:     mov    %edi,-0x14(%rbp)
   0x0000000000400936 <+11>:    mov    %rsi,-0x20(%rbp)
   0x000000000040093a <+15>:    mov    %fs:0x28,%rax
   0x0000000000400943 <+24>:    mov    %rax,-0x8(%rbp)
   0x0000000000400947 <+28>:    xor    %eax,%eax
   0x0000000000400949 <+30>:    mov    $0x0,%edi
   0x000000000040094e <+35>:    call   0x400750 <time@plt>
   0x0000000000400953 <+40>:    mov    %eax,%edi
   0x0000000000400955 <+42>:    call   0x400730 <srand@plt>
   0x000000000040095a <+47>:    call   0x400790 <rand@plt>
   0x000000000040095f <+52>:    mov    %eax,-0xc(%rbp)
   0x0000000000400962 <+55>:    lea    0x1c7(%rip),%rdi        # 0x400b30
   0x0000000000400969 <+62>:    call   0x4006e0 <puts@plt>
   0x000000000040096e <+67>:    lea    0x1e3(%rip),%rdi        # 0x400b58
   0x0000000000400975 <+74>:    call   0x4006e0 <puts@plt>
   0x000000000040097a <+79>:    lea    0x207(%rip),%rdi        # 0x400b88
   0x0000000000400981 <+86>:    call   0x4006e0 <puts@plt>
   0x0000000000400986 <+91>:    lea    0x21b(%rip),%rdi        # 0x400ba8
   0x000000000040098d <+98>:    mov    $0x0,%eax
   0x0000000000400992 <+103>:   call   0x400710 <printf@plt>
   0x0000000000400997 <+108>:   mov    0x2006ea(%rip),%rax        # 0x601088 <stdout@@GLIBC_2.2.5>
   0x000000000040099e <+115>:   mov    %rax,%rdi
   0x00000000004009a1 <+118>:   call   0x400760 <fflush@plt>
   0x00000000004009a6 <+123>:   lea    -0x10(%rbp),%rax
   0x00000000004009aa <+127>:   mov    %rax,%rsi
   0x00000000004009ad <+130>:   lea    0x208(%rip),%rdi        # 0x400bbc
   0x00000000004009b4 <+137>:   mov    $0x0,%eax
   0x00000000004009b9 <+142>:   call   0x400780 <__isoc99_scanf@plt>
   0x00000000004009be <+147>:   mov    -0x10(%rbp),%eax
   0x00000000004009c1 <+150>:   mov    %eax,%esi
   0x00000000004009c3 <+152>:   lea    0x1f5(%rip),%rdi        # 0x400bbf
   0x00000000004009ca <+159>:   mov    $0x0,%eax
--Type <RET> for more, q to quit, c to continue without paging--RET
   0x00000000004009cf <+164>:   call   0x400710 <printf@plt>
   0x00000000004009d4 <+169>:   mov    -0xc(%rbp),%eax
   0x00000000004009d7 <+172>:   mov    %eax,%esi
   0x00000000004009d9 <+174>:   lea    0x1f3(%rip),%rdi        # 0x400bd3
   0x00000000004009e0 <+181>:   mov    $0x0,%eax
   0x00000000004009e5 <+186>:   call   0x400710 <printf@plt>
   0x00000000004009ea <+191>:   mov    0x200697(%rip),%rax        # 0x601088 <stdout@@GLIBC_2.2.5>
   0x00000000004009f1 <+198>:   mov    %rax,%rdi
   0x00000000004009f4 <+201>:   call   0x400760 <fflush@plt>
   0x00000000004009f9 <+206>:   mov    -0x10(%rbp),%eax
   0x00000000004009fc <+209>:   cmp    %eax,-0xc(%rbp)
   0x00000000004009ff <+212>:   jne    0x400a14 <main+233>
   0x0000000000400a01 <+214>:   lea    0x1e0(%rip),%rdi        # 0x400be8
   0x0000000000400a08 <+221>:   call   0x4006e0 <puts@plt>
   0x0000000000400a0d <+226>:   call   0x400877 <giveFlag>
   0x0000000000400a12 <+231>:   jmp    0x400a20 <main+245>
   0x0000000000400a14 <+233>:   lea    0x1fd(%rip),%rdi        # 0x400c18
   0x0000000000400a1b <+240>:   call   0x4006e0 <puts@plt>
   0x0000000000400a20 <+245>:   mov    0x200661(%rip),%rax        # 0x601088 <stdout@@GLIBC_2.2.5>
   0x0000000000400a27 <+252>:   mov    %rax,%rdi
   0x0000000000400a2a <+255>:   call   0x400760 <fflush@plt>
   0x0000000000400a2f <+260>:   mov    $0x0,%eax
   0x0000000000400a34 <+265>:   mov    -0x8(%rbp),%rdx
   0x0000000000400a38 <+269>:   xor    %fs:0x28,%rdx
   0x0000000000400a41 <+278>:   je     0x400a48 <main+285>
   0x0000000000400a43 <+280>:   call   0x400700 <__stack_chk_fail@plt>
   0x0000000000400a48 <+285>:   leave
   0x0000000000400a49 <+286>:   ret
End of assembler dump.
(gdb) break *0x40095f
Breakpoint 1 at 0x40095f
(gdb) run
Starting program: /mnt/c/users/varsha/downloads/time
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x000000000040095f in main ()
(gdb) print $eax
$1 = 2001081298
(gdb) continue
Continuing.
Welcome to the number guessing game!
I'm thinking of a number. Can you guess it?
Guess right and you get a flag!
Enter your number: 2001081298
Your guess was 2001081298.
Looking for 2001081298.
You won. Guess was right! Here's your flag:
Flag file not found!  Contact an H3 admin for assistance.
[Inferior 1 (process 4213) exited normally]
(gdb)
```


```
vasha@Varsha:/mnt/c/users/varsha/downloads$ sudo mkdir -p /home/h3
[sudo] password for vasha:
Sorry, try again.
[sudo] password for vasha:
vasha@Varsha:/mnt/c/users/varsha/downloads$ sudo mkdir -p /home/h3
vasha@Varsha:/mnt/c/users/varsha/downloads$ sudo echo "H3{test_flag}" > /home/h3/flag.txt
-bash: /home/h3/flag.txt: Permission denied
vasha@Varsha:/mnt/c/users/varsha/downloads$ sudo chmod 644 /home/h3/flag.txt
chmod: cannot access '/home/h3/flag.txt': No such file or directory
vasha@Varsha:/mnt/c/users/varsha/downloads$ echo "H3{test_flag}" | sudo tee /home/h3/flag.txt
H3{test_flag}
vasha@Varsha:/mnt/c/users/varsha/downloads$ sudo chmod 644 /home/h3/flag.txt
vasha@Varsha:/mnt/c/users/varsha/downloads$ gdb ./time
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
Reading symbols from ./time...

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
Downloading separate debug info for /mnt/c/users/varsha/downloads/time
(No debugging symbols found in ./time)
(gdb)  break *0x40095f
Breakpoint 1 at 0x40095f
(gdb) run
Starting program: /mnt/c/users/varsha/downloads/time
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x000000000040095f in main ()
(gdb) print $eax
$1 = 343086080
(gdb) continue
Continuing.
Welcome to the number guessing game!
I'm thinking of a number. Can you guess it?
Guess right and you get a flag!
Enter your number: 343086080
Your guess was 343086080.
Looking for 343086080.
You won. Guess was right! Here's your flag:
H3{test_flag}

[Inferior 1 (process 4331) exited normally]
(gdb)
```



## VeridisQuo

### Solution:

I first checked the file's type and unzipped it before running `apktool d VeridisQuo.apk -o VeridisQuo_src` to get the APK. Then I used `jadx` to decompile the code to see what was going on. This is where I found 28 parts of the flag. Then I found out how the parts were structured in `resources<res<layout<activity_main.xml` then plotted the points to get the flag.

<img width="1374" height="919" alt="image" src="https://github.com/user-attachments/assets/f5d7edda-ddb9-4aa2-b67d-54d6925fb5db" />

<img width="1606" height="1108" alt="image" src="https://github.com/user-attachments/assets/938d40e5-2432-4c31-bcb2-8b2837089250" />

### Flag:
**Flag:** `byuctf{android_piece_0f_c4ke}`

## dusty

### Solution:

1. dust_noob
   


