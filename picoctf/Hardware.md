# 1. IQ Test

let your input x = 30478191278.

wrap your answer with nite{ } for the flag.

As an example, entering x = 34359738368 gives (y0, ..., y11), so the flag would be nite{010000000011}.

## Solution:

- I first converted x into binary, then substituted them for the values `{x0, x1, .... , x35}`. Then I manually solved the logic gates.

## Flag:

`nite{100010011000}`

## Resources:

- https://www.rapidtables.com/convert/number/decimal-to-binary.html?x=30478191278



# 2. i like logic

i like logic and i like files, apparently, they have something in common, what should my next step be.

## Solution:

- I loaded it into Logic 2 because I saw a `.json` file. I used ASYNC to find the flag:

<img width="1901" height="1115" alt="image" src="https://github.com/user-attachments/assets/3a68871d-a3d5-4b10-a621-95b0f661bab0" />



## Flag:

`FCSC{b1dee4eeadf6c4e60aeb142b0b486344e64b12b40d1046de95c89ba5e23a9925}`



# 3. bare metal alchemist
my friend recommended me this anime but i think i've heard a wrong name.

## Solution

- The file given was a `.elf` file (compiled for AVR microcontrollers), so I used `avr` to analyze the file.
- `avr-objdump -m avr -D firmware.elf` revealed the assembly code where the data was stored
- The dissassembly revealed that XOR encryption was being used, Key = 0xA5, Encoded data starts at address 0x68
- I found the raw bytes:
```
f1 e3 e6 e6 f1 e3 de f1 cd 94 d6 fa 94 d6 fa d6 
ca c8 96 fa d6 94 c8 d5 c9 96 fa 91 d7 c1 d0 94 
cb ca fa c3 94 d7 c8 d2 91 d7 c0 d8
```

- Then I used this python code to find the flag:
```
encoded_data = [
    0xf1, 0xe3, 0xe6, 0xe6, 0xf1, 0xe3, 0xde, 0xf1, 0xcd, 0x94, 0xd6, 0xfa,
    0x94, 0xd6, 0xfa, 0xd6, 0xca, 0xc8, 0x96, 0xfa, 0xd6, 0x94, 0xc8, 0xd5,
    0xc9, 0x96, 0xfa, 0x91, 0xd7, 0xc1, 0xd0, 0x94, 0xcb, 0xca, 0xfa, 0xc3,
    0x94, 0xd7, 0xc8, 0xd2, 0x91, 0xd7, 0xc0, 0xd8
]

key = 0xA5

decoded = bytes([byte ^ key for byte in encoded_data])
print("Decoded data:", decoded)
print("As string:", decoded.decode('ascii'))
```

## Flag:

`TFCCTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}`

## Concepts Learned:

I learned more about `.elf` files.

## Resources

- https://ccrma.stanford.edu/planetccrma/man/man1/avr-objdump.1.html
  
