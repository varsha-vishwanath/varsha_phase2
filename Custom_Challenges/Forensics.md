# Digital Forensics

## 1. Hide and Seek

### Description:
Sakamoto’s at it again with a game of hide and seek, but this time, it’s not with Shin or his daughter. An old friend hid some secret data in this image. Can you find it before the others do?

Hint: Even in retirement, Sakamoto never loses at hide and seek. Maybe stegseek can help you keep up.

### Solution:
I used `steghide` and `stegseek` to find the password for `sakamoto.jpg`. I downloaded `rockyou.txt` (The password list) from a website online and used it when using stegseek. 
After that I just catted the flag. 

This challenge took me a bit because I originally used an incomplete password list and was unable to get the password.

<img width="1832" height="1006" alt="hide" src="https://github.com/user-attachments/assets/aae93297-80ae-4fd5-8d39-358a15e06069" />

### Flag:

**Flag:** `nite{h1d3_4nd_s33k_but_w1th_st3g_sdfu9s8}`



## Nutrela Chunks

### Description:
One of my favorite foods is soya chunks. But as I was enjoying some Nutrela today, I noticed a few chunks weren’t quite right. Seems like something’s off with their structure. Could you help me fix these broken chunks so I can enjoy my meal again?

### Solution
The original `nutrela.png` file seemed to be corrupted so i used `hexedit` to go in and change the magic bytes to the correct `.png` format. Then I kept using `pngcheck` to figure out what was wrong with the file and changed the bytes in hexedit accordingly. Finally, I opened the image to get the flag.

<img width="1000" height="852" alt="nutrela" src="https://github.com/user-attachments/assets/75c3e825-953d-411c-94ab-acaa1eb257f8" />

<img width="1000" height="1000" alt="nutrela" src="https://github.com/user-attachments/assets/f7e69410-e734-4efd-8730-bfd00cc3f073" />

### Flag:
**Flag:** `nite{n0w_y0u_kn0w_ab0ut_PNG_chunk5}`



## 

