# 1. Trivial Flag Transfer Protocol
Figure out how they moved the flag.

## Solution:
- I first used my wsl to get wireshark and open the file. Since the challenge is about tftp, I first started looking into the file with that as the filter:
<img width="1642" height="716" alt="image" src="https://github.com/user-attachments/assets/38dd6dae-cb09-4c17-b54c-3b9bdc4b4c2b" />

- Then, I exported the files shown and opened the `instructions` file, thinking i'd get some direction, to find a bunch of encrypted text. I used an ROT13 cipher website to decode the message:
<img width="959" height="859" alt="image" src="https://github.com/user-attachments/assets/99c42778-a1db-440c-9ec1-27cab832eb3a" />

- Then my next move was to look at `plan`, where I found:
<img width="954" height="849" alt="image" src="https://github.com/user-attachments/assets/4f6e455e-2cd7-46f9-b8e0-fa313ebd4d5b" />

- That told me `DUEDILIGENCE` was some kind of password to the flag and it was hidden using the file `program.deb`. I had to look into the pictures to find the flag.
- I figured `program.deb` would tell me more so I tried finding more about what it does:
  
  ```
  vasha@Varsha:/mnt/c/users/varsha/desktop/cryptonite/tp2/tftp$ dpkg -I program.deb
   new Debian package, version 2.0.
   size 138310 bytes: control archive=1250 bytes.
       826 bytes,    18 lines      control
      1184 bytes,    17 lines      md5sums
   Package: steghide
   Source: steghide (0.5.1-9.1)
   Version: 0.5.1-9.1+b1
   Architecture: amd64
   Maintainer: Ola Lundqvist <opal@debian.org>
   Installed-Size: 426
   Depends: libc6 (>= 2.2.5), libgcc1 (>= 1:4.1.1), libjpeg62-turbo (>= 1:1.3.1), libmcrypt4, libmhash2, libstdc++6 (>= 4.9), zlib1g (>= 1:1.1.4)
   Section: misc
   Priority: optional
   Description: A steganography hiding tool
    Steghide is steganography program which hides bits of a data file
    in some of the least significant bits of another file in such a way
    that the existence of the data file is not visible and cannot be proven.
    .
    Steghide is designed to be portable and configurable and features hiding
    data in bmp, wav and au files, blowfish encryption, MD5 hashing of
    passphrases to blowfish keys, and pseudo-random distribution of hidden bits
    in the container data.
    ```
- This told me that program.deb uses steganography to hide the information in the images so I used an online steganography decoder in order to retrieve the flag from the images. (I tried all the images till I got it at picture3)
<img width="631" height="323" alt="image" src="https://github.com/user-attachments/assets/05a7826b-fdb0-495b-95fd-5475338c695c" />
 
## Flag:

`picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}`

## Concepts Learned:
- I learned how to inspect TFTP traffic in Wireshark and export TFTP objects.
- I learned how to inspect a .deb file with dpkg -I to discover what program was bundled.
- I learned the basics of steganography with steghide (how to extract hidden files using a passphrase).


## Notes

I originally tried using steghide to decode the images instead of a generic website because that was what the descritption mentioned. However, I kept running into errors and figured it would be easier to use a website

## Resources
- https://rot13.com/
- https://www.reddit.com/r/linux4noobs/comments/uwpme1/what_is_the_difference_between_a_deb_file_and_a/#:~:text=What%20is%20the%20difference%20between,Alexander%2D369
- https://futureboy.us/stegano/decinput.html



# 2. tunn3l v1s10n
We found this file. Recover the flag.

## Solution
- When I first looked into the file, it said it was a generic data file. But this reminded me of the `M1D1` challenge from the citadel ctf so I decided to look into the hexdump of the file:
  ```
  vasha@Varsha:/mnt/c/users/varsha/downloads$ file tunn3l_v1s10n
  tunn3l_v1s10n: data
  vasha@Varsha:/mnt/c/users/varsha/downloads$ hexdump -C tunn3l_v1s10n | head
  00000000  42 4d 8e 26 2c 00 00 00  00 00 ba d0 00 00 ba d0  |BM.&,...........|
  00000010  00 00 6e 04 00 00 32 01  00 00 01 00 18 00 00 00  |..n...2.........|
  00000020  00 00 58 26 2c 00 25 16  00 00 25 16 00 00 00 00  |..X&,.%...%.....|
  00000030  00 00 00 00 00 00 23 1a  17 27 1e 1b 29 20 1d 2a  |......#..'..) .*|
  00000040  21 1e 26 1d 1a 31 28 25  35 2c 29 33 2a 27 38 2f  |!.&..1(%5,)3*'8/|
  00000050  2c 2f 26 23 33 2a 26 2d  24 20 3b 32 2e 32 29 25  |,/&#3*&-$ ;2.2)%|
  00000060  30 27 23 33 2a 26 38 2c  28 36 2b 27 39 2d 2b 2f  |0'#3*&8,(6+'9-+/|
  00000070  26 23 1d 12 0e 23 17 11  29 16 0e 55 3d 31 97 76  |&#...#..)..U=1.v|
  00000080  66 8b 66 52 99 6d 56 9e  70 58 9e 6f 54 9c 6f 54  |f.fR.mV.pX.oT.oT|
  00000090  ab 7e 63 ba 8c 6d bd 8a  69 c8 97 71 c1 93 71 c1  |.~c..m..i..q..q.|
  ```
- The magic bits told me that this was a `.bmp` file but changing the extension didn't work, so I tried looking into what else could be wrong with the file.
- I found an explanation of `.bmp` hexmaps and realized firstly realized that the data offset bytes weren't formatted by least significant bit first (it was  `BA D0  00 00`). So I changed that to `00 00 D0 BA`. But that didn't get me anywhere
- Then I saw that the infoheader bytes were given incorrectly (they were the same as the data offset bytes and did not equal 40). So I used hexedit to change that: ` 42 4D 8E 26  2C 00 00 00  00 00 BA D0  00 00 28 00  00 00`.
- Now here's what the image looked like:
<img width="1156" height="331" alt="image" src="https://github.com/user-attachments/assets/6089f5ad-4a56-4b43-bee1-fec03afc63d6" />
- It looked very obviously cropped so I looked into how I could edit the size and ended up playing with the height bytes (converting to decimal, multiplying them by different scales, conversting back to hex and writing in little endian form in hexedit). I finally settled for editing to `20 03 00 00`. Which increased the height to 800 pixels. It only gave me a half view of the flag but it was enough to decipher it:
<img width="1160" height="830" alt="image" src="https://github.com/user-attachments/assets/0fd03aea-193b-4c21-8894-7b58029e82cf" />

## Flag:
`picoCTF{qu1t3_a_v13w_2020}`

## Concepts learned
- I learned how to decipher a hexdump and learned its structure for .bmp files

## Notes
- It took me a LONG time to finally settle on the height. I couldn't find an easier way to find the heigh of the actual image.

## Resources
- https://en.wikipedia.org/wiki/List_of_file_signatures
- https://www.ece.ualberta.ca/~elliott/ee552/studentAppNotes/2003_w/misc/bmp_file_format/bmp_file_format.htm
- https://www.rapidtables.com/convert/number/decimal-to-hex.html?x=800



# 3. m00nwalk
