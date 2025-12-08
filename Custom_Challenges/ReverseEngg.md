# Reverse Engineering

## JoyDivision

### Solution:

I used ghidra to decompile `disorder` and found that the original secret was being encrypted. The bits on `palatinepackflag.txt` were flipped and expanded thrice before being placed in the given `flag.txt`. I used a python script to reverse this logic which allowed me to find the flag.

```
vasha@Varsha:/mnt/c/users/varsha/downloads$ ./disorder

May Jupiter strike you down Caeser before you seize the treasury!! You will have to tear me apart
for me to tell you the flag to unlock the Roman Treasury and fund your civil war. I, Lucius Caecilius
Metellus, shall not let you pass until you get this password right. (or threaten to kill me-)
```

<img width="1912" height="888" alt="image" src="https://github.com/user-attachments/assets/3b9cf447-8dce-45a0-aa0c-a0a2673e045e" />

```
def collapse(s: bytes) -> bytearray:
    res = bytearray()
    for i in range(len(s) // 2):
        if i % 2 == 0:
            res.append((s[2 * i] & 0x0F) | (s[2 * i + 1] & 0xF0))
        else:
            res.append((s[2 * i] & 0xF0) | (s[2 * i + 1] & 0x0F))
    return res

def main():
    data = open("flag.txt", "rb").read()

    data = collapse(data)
    data = collapse(data)
    data = collapse(data)

    data = bytearray(data)
    v3 = 105
    for i in range(len(data)):
        if i % 2 == 0:
            data[i] = (~data[i]) & 0xFF
        else:
            data[i] ^= v3 & 0xFF
            v3 += 32

    print(data.decode(errors="ignore"))

if __name__ == "__main__":
    main()
```

### Flag:
**Flag:** `sunshine{C3A5ER_CR055ED_TH3_RUB1C0N}`



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

1. **dust_noob**
   I tried running the binary, and then dissassembled it using `gdb`. I found where the main function was and dissassembled that and found a set of `movb` instructions that    was storing some bytes. The program was loading each byte from the array and XORing it with `0x3f` to get the flag. So I did exactly that and got the first flag.

```
   vasha@Varsha:/mnt/c/users/varsha/downloads$ file dust_noob
dust_noob: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0e91c3247b82497c448e69060be605936f5f06fe, for GNU/Linux 3.2.0, not stripped
vasha@Varsha:/mnt/c/users/varsha/downloads$ ./dust_noob
Shiny Clean™ Rust Remover Budget Edition! Looks like you didn't win this time! Try again?
vasha@Varsha:/mnt/c/users/varsha/downloads$ gdb ./dust_noob
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
Reading symbols from ./dust_noob...

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
Downloading separate debug info for /mnt/c/users/varsha/downloads/dust_noob
(No debugging symbols found in ./dust_noob)
warning: Missing auto-load script at offset 0 in section .debug_gdb_scripts
of file /mnt/c/users/varsha/downloads/dust_noob.
Use `info auto-load python-scripts [REGEXP]' to list them.
(gdb) dissassemble main
Undefined command: "dissassemble".  Try "help".
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000000000007d70 <+0>:     push   %rax
   0x0000000000007d71 <+1>:     mov    %rsi,%rdx
   0x0000000000007d74 <+4>:     lea    0x432e8(%rip),%rax        # 0x4b063 <__rustc_debug_gdb_scripts_section__>
   0x0000000000007d7b <+11>:    mov    (%rax),%al
   0x0000000000007d7d <+13>:    movslq %edi,%rsi
   0x0000000000007d80 <+16>:    lea    -0x247(%rip),%rdi        # 0x7b40 <_ZN10shinyclean4main17h4b15dd54e331d693E>
   0x0000000000007d87 <+23>:    xor    %ecx,%ecx
   0x0000000000007d89 <+25>:    call   0x7ac0 <_ZN3std2rt10lang_start17h91ff47afc442db24E>
   0x0000000000007d8e <+30>:    pop    %rcx
   0x0000000000007d8f <+31>:    ret
End of assembler dump.
(gdb) disassemble 0x7b40
Dump of assembler code for function _ZN10shinyclean4main17h4b15dd54e331d693E:
   0x0000000000007b40 <+0>:     sub    $0x108,%rsp
   0x0000000000007b47 <+7>:     lea    0x2a(%rsp),%rdi
   0x0000000000007b4c <+12>:    xor    %esi,%esi
   0x0000000000007b4e <+14>:    mov    $0x17,%edx
   0x0000000000007b53 <+19>:    call   0x6050 <memset@plt>
   0x0000000000007b58 <+24>:    movb   $0x7b,0x41(%rsp)
   0x0000000000007b5d <+29>:    movb   $0x5e,0x42(%rsp)
   0x0000000000007b62 <+34>:    movb   $0x48,0x43(%rsp)
   0x0000000000007b67 <+39>:    movb   $0x58,0x44(%rsp)
   0x0000000000007b6c <+44>:    movb   $0x7c,0x45(%rsp)
   0x0000000000007b71 <+49>:    movb   $0x6b,0x46(%rsp)
   0x0000000000007b76 <+54>:    movb   $0x79,0x47(%rsp)
   0x0000000000007b7b <+59>:    movb   $0x44,0x48(%rsp)
   0x0000000000007b80 <+64>:    movb   $0x79,0x49(%rsp)
   0x0000000000007b85 <+69>:    movb   $0x6d,0x4a(%rsp)
   0x0000000000007b8a <+74>:    movb   $0xc,0x4b(%rsp)
   0x0000000000007b8f <+79>:    movb   $0xc,0x4c(%rsp)
   0x0000000000007b94 <+84>:    movb   $0x60,0x4d(%rsp)
   0x0000000000007b99 <+89>:    movb   $0x7c,0x4e(%rsp)
   0x0000000000007b9e <+94>:    movb   $0xb,0x4f(%rsp)
   0x0000000000007ba3 <+99>:    movb   $0x6d,0x50(%rsp)
   0x0000000000007ba8 <+104>:   movb   $0x60,0x51(%rsp)
   0x0000000000007bad <+109>:   movb   $0x68,0x52(%rsp)
   0x0000000000007bb2 <+114>:   movb   $0xb,0x53(%rsp)
   0x0000000000007bb7 <+119>:   movb   $0xa,0x54(%rsp)
   0x0000000000007bbc <+124>:   movb   $0x77,0x55(%rsp)
   0x0000000000007bc1 <+129>:   movb   $0x1e,0x56(%rsp)
   0x0000000000007bc6 <+134>:   movb   $0x42,0x57(%rsp)
   0x0000000000007bcb <+139>:   movq   $0x0,0x58(%rsp)
   0x0000000000007bd4 <+148>:   mov    0x58(%rsp),%rax
   0x0000000000007bd9 <+153>:   mov    %rax,0x20(%rsp)
   0x0000000000007bde <+158>:   cmp    $0x17,%rax
   0x0000000000007be2 <+162>:   jae    0x7c03 <_ZN10shinyclean4main17h4b15dd54e331d693E+195>
   0x0000000000007be4 <+164>:   mov    0x20(%rsp),%rax
   0x0000000000007be9 <+169>:   mov    0x41(%rsp,%rax,1),%al
--Type <RET> for more, q to quit, c to continue without paging--RET
   0x0000000000007bed <+173>:   mov    %al,0x17(%rsp)
   0x0000000000007bf1 <+177>:   mov    0x58(%rsp),%rax
   0x0000000000007bf6 <+182>:   mov    %rax,0x18(%rsp)
   0x0000000000007bfb <+187>:   cmp    $0x17,%rax
   0x0000000000007bff <+191>:   jb     0x7c1d <_ZN10shinyclean4main17h4b15dd54e331d693E+221>
   0x0000000000007c01 <+193>:   jmp    0x7c42 <_ZN10shinyclean4main17h4b15dd54e331d693E+258>
   0x0000000000007c03 <+195>:   mov    0x20(%rsp),%rdi
   0x0000000000007c08 <+200>:   lea    0x4c969(%rip),%rdx        # 0x54578
   0x0000000000007c0f <+207>:   lea    -0x8e1(%rip),%rax        # 0x7335 <_ZN4core9panicking18panic_bounds_check17h8307ccead484a122E>
   0x0000000000007c16 <+214>:   mov    $0x17,%esi
   0x0000000000007c1b <+219>:   call   *%rax
   0x0000000000007c1d <+221>:   mov    0x18(%rsp),%rax
   0x0000000000007c22 <+226>:   mov    0x17(%rsp),%cl
   0x0000000000007c26 <+230>:   xor    $0x3f,%cl
   0x0000000000007c29 <+233>:   mov    %cl,0x2a(%rsp,%rax,1)
   0x0000000000007c2d <+237>:   mov    0x58(%rsp),%rax
   0x0000000000007c32 <+242>:   add    $0x1,%rax
   0x0000000000007c36 <+246>:   mov    %rax,0x8(%rsp)
   0x0000000000007c3b <+251>:   setb   %al
   0x0000000000007c3e <+254>:   jb     0x7c73 <_ZN10shinyclean4main17h4b15dd54e331d693E+307>
   0x0000000000007c40 <+256>:   jmp    0x7c5c <_ZN10shinyclean4main17h4b15dd54e331d693E+284>
   0x0000000000007c42 <+258>:   mov    0x18(%rsp),%rdi
   0x0000000000007c47 <+263>:   lea    0x4c942(%rip),%rdx        # 0x54590
   0x0000000000007c4e <+270>:   lea    -0x920(%rip),%rax        # 0x7335 <_ZN4core9panicking18panic_bounds_check17h8307ccead484a122E>
   0x0000000000007c55 <+277>:   mov    $0x17,%esi
   0x0000000000007c5a <+282>:   call   *%rax
   0x0000000000007c5c <+284>:   mov    0x8(%rsp),%rax
   0x0000000000007c61 <+289>:   mov    %rax,0x58(%rsp)
   0x0000000000007c66 <+294>:   cmpq   $0x17,0x58(%rsp)
   0x0000000000007c6c <+300>:   je     0x7c83 <_ZN10shinyclean4main17h4b15dd54e331d693E+323>
   0x0000000000007c6e <+302>:   jmp    0x7bd4 <_ZN10shinyclean4main17h4b15dd54e331d693E+148>
   0x0000000000007c73 <+307>:   lea    0x4c92e(%rip),%rdi        # 0x545a8
   0x0000000000007c7a <+314>:   lea    -0x461(%rip),%rax        # 0x7820 <_ZN4core9panicking11panic_const24panic_const_add_overflow17hf2f4fb688348b3b0E>
--Type <RET> for more, q to quit, c to continue without paging--q
Quit
(gdb)
```

Byte 1: 0x7b ⊕ 0x3f = 0x44 = 'D'
Byte 2: 0x5e ⊕ 0x3f = 0x61 = 'a'
Byte 3: 0x48 ⊕ 0x3f = 0x77 = 'w'
Byte 4: 0x58 ⊕ 0x3f = 0x67 = 'g'
Byte 5: 0x7c ⊕ 0x3f = 0x43 = 'C'
Byte 6: 0x6b ⊕ 0x3f = 0x54 = 'T'
Byte 7: 0x79 ⊕ 0x3f = 0x46 = 'F'
Byte 8: 0x44 ⊕ 0x3f = 0x7b = '{'
Byte 9: 0x79 ⊕ 0x3f = 0x46 = 'F'
Byte 10: 0x6d ⊕ 0x3f = 0x52 = 'R'
Byte 11: 0x0c ⊕ 0x3f = 0x33 = '3'
Byte 12: 0x0c ⊕ 0x3f = 0x33 = '3'
Byte 13: 0x60 ⊕ 0x3f = 0x5f = '_'
Byte 14: 0x7c ⊕ 0x3f = 0x43 = 'C'
Byte 15: 0x0b ⊕ 0x3f = 0x34 = '4'
Byte 16: 0x6d ⊕ 0x3f = 0x52 = 'R'
Byte 17: 0x60 ⊕ 0x3f = 0x5f = '_'
Byte 18: 0x68 ⊕ 0x3f = 0x57 = 'W'
Byte 19: 0x0b ⊕ 0x3f = 0x34 = '4'
Byte 20: 0x0a ⊕ 0x3f = 0x35 = '5'
Byte 21: 0x77 ⊕ 0x3f = 0x48 = '8'
Byte 22: 0x1e ⊕ 0x3f = 0x21 = '!'
Byte 23: 0x42 ⊕ 0x3f = 0x7d = '}'


**Flag:** `DawgCTF{FR33_C4R_W458!}`

2. **dust_intermediate**
    I ran the binary and saw it wanted a "challenge phrase". I used gdb to find the real Rust main function at `0xd430`, then opened it in `Ghidra`. In the decompilation of     `shinyclean2::main`, I found 21 bytes stored on the stack:

    ```
    0xEA 0xD9 0x31 0x22 0xD3 0xE6 0x97 0x70 0x16 0xA2 0xA8 0x1B 0x61 0xFC 0x76 0x68 0x7B 0xAB 0xB8 0x27 0x96
    ```

    The program reads input, spawns a thread that XORs each byte with `0xFF`, then compares the result to those bytes. So I just did the reverse: XOR each target byte with     `0xFF` to get the input.

    ```
    target = bytes([0xEA, 0xD9, 0x31, 0x22, 0xD3, 0xE6, 0x97, 0x70, 0x16, 0xA2, 0xA8, 0x1B, 0x61, 0xFC, 0x76, 0x68, 0x7B, 0xAB, 0xB8, 0x27, 0x96])
    input_bytes = bytes([b ^ 0xFF for b in target])
    print(input_bytes)  # This is the challenge phrase
    ```

    Feeding this to the binary gave the win message.

   <img width="1916" height="1145" alt="image" src="https://github.com/user-attachments/assets/50b6833d-6e51-4194-91f9-5a878b87f75e" />

   **Flag:** `DawgCTF{S0000_CL43N!}`

4. **dust_pro**
   Running the binary showed that it expected a code in return for the flag so I used gdb to disassemble the main and found that `shinyclean::main` is the real entry point.    Then I found that 25 bytes are being stored on stack in assembly. Then I located the expected `SHA` hash at address `0x5b134` in `.rodata` section. Then I found XOR         logic at address `0x9b67` : `xor 0xb7(%rsp,%rax,1),%cl`. So the program decrypts the bytes and compares it to make sure that `SHA-256` hash of the same matches the          given hash. From whwihc i got `code = 139 + (104<<8) + (105<<16) + (212<<24)`. That helped me get the flag.

   

   
<img width="1240" height="227" alt="image" src="https://github.com/user-attachments/assets/bcc3ddc7-20b6-4083-bc70-744d9f54f1e9" />

```
(gdb) x/30i 0x9b0f
   0x9b0f <_ZN11shinyclean24main17h38206fcee08f84d4E+1295>:     mov    0x28(%rsp),%rax
   0x9b14 <_ZN11shinyclean24main17h38206fcee08f84d4E+1300>:     and    $0x3,%rax
   0x9b18 <_ZN11shinyclean24main17h38206fcee08f84d4E+1304>:     mov    %rax,0x8(%rsp)
   0x9b1d <_ZN11shinyclean24main17h38206fcee08f84d4E+1309>:     cmp    $0x4,%rax
   0x9b21 <_ZN11shinyclean24main17h38206fcee08f84d4E+1313>:
    jae    0x9b40 <_ZN11shinyclean24main17h38206fcee08f84d4E+1344>
   0x9b23 <_ZN11shinyclean24main17h38206fcee08f84d4E+1315>:     mov    0x28(%rsp),%rax
   0x9b28 <_ZN11shinyclean24main17h38206fcee08f84d4E+1320>:     mov    0x8(%rsp),%rcx
   0x9b2d <_ZN11shinyclean24main17h38206fcee08f84d4E+1325>:     mov    0x124(%rsp,%rcx,1),%cl
   0x9b34 <_ZN11shinyclean24main17h38206fcee08f84d4E+1332>:     mov    %cl,0x7(%rsp)
   0x9b38 <_ZN11shinyclean24main17h38206fcee08f84d4E+1336>:     cmp    $0x19,%rax
   0x9b3c <_ZN11shinyclean24main17h38206fcee08f84d4E+1340>:
    jb     0x9b5e <_ZN11shinyclean24main17h38206fcee08f84d4E+1374>
   0x9b3e <_ZN11shinyclean24main17h38206fcee08f84d4E+1342>:
    jmp    0x9b7a <_ZN11shinyclean24main17h38206fcee08f84d4E+1402>
   0x9b40 <_ZN11shinyclean24main17h38206fcee08f84d4E+1344>:     mov    0x8(%rsp),%rdi
   0x9b45 <_ZN11shinyclean24main17h38206fcee08f84d4E+1349>:     lea    0x64dac(%rip),%rdx        # 0x6e8f8
   0x9b4c <_ZN11shinyclean24main17h38206fcee08f84d4E+1356>:
    lea    -0x17be(%rip),%rax        # 0x8395 <_ZN4core9panicking18panic_bounds_check17h8307ccead484a122E>
   0x9b53 <_ZN11shinyclean24main17h38206fcee08f84d4E+1363>:     mov    $0x4,%esi
   0x9b58 <_ZN11shinyclean24main17h38206fcee08f84d4E+1368>:     call   *%rax
   0x9b5a <_ZN11shinyclean24main17h38206fcee08f84d4E+1370>:
    jmp    0x9b5c <_ZN11shinyclean24main17h38206fcee08f84d4E+1372>
   0x9b5c <_ZN11shinyclean24main17h38206fcee08f84d4E+1372>:     ud2
   0x9b5e <_ZN11shinyclean24main17h38206fcee08f84d4E+1374>:     mov    0x28(%rsp),%rax
   0x9b63 <_ZN11shinyclean24main17h38206fcee08f84d4E+1379>:     mov    0x7(%rsp),%cl
   0x9b67 <_ZN11shinyclean24main17h38206fcee08f84d4E+1383>:     xor    0xb7(%rsp,%rax,1),%cl
   0x9b6e <_ZN11shinyclean24main17h38206fcee08f84d4E+1390>:     mov    %cl,0xb7(%rsp,%rax,1)
   0x9b75 <_ZN11shinyclean24main17h38206fcee08f84d4E+1397>:
    jmp    0x98c9 <_ZN11shinyclean24main17h38206fcee08f84d4E+713>
   0x9b7a <_ZN11shinyclean24main17h38206fcee08f84d4E+1402>:     mov    0x28(%rsp),%rdi
   0x9b7f <_ZN11shinyclean24main17h38206fcee08f84d4E+1407>:     lea    0x64d8a(%rip),%rdx        # 0x6e910
   0x9b86 <_ZN11shinyclean24main17h38206fcee08f84d4E+1414>:
    lea    -0x17f8(%rip),%rax        # 0x8395 <_ZN4core9panicking18panic_bounds_check17h8307ccead484a122E>
--Type <RET> for more, q to quit, c to continue without paging--quit
Quit
```

<img width="550" height="71" alt="image" src="https://github.com/user-attachments/assets/3da580dd-a1d7-4cfe-bd7e-967d42d763a6" />

**Flag:** `DawgCTF{4LL_RU57_N0_C4R!}`




   


