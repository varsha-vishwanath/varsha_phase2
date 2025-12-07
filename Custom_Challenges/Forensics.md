# Digital Forensics

## 1. Hide and Seek

### Description
Sakamoto’s at it again with a game of hide and seek, but this time, it’s not with Shin or his daughter. An old friend hid some secret data in this image. Can you find it before the others do?

Hint: Even in retirement, Sakamoto never loses at hide and seek. Maybe stegseek can help you keep up.

### Solution
I used `steghide` and `stegseek` to find the password for `sakamoto.jpg`. I downloaded `rockyou.txt` (The password list) from a website online and used it when using stegseek. 
After that I just catted the flag. 

This challenge took me a bit because I originally used an incomplete password list and was unable to get the password.

<img width="1832" height="1006" alt="hide" src="https://github.com/user-attachments/assets/aae93297-80ae-4fd5-8d39-358a15e06069" />

### Flag

**Flag:** `nite{h1d3_4nd_s33k_but_w1th_st3g_sdfu9s8}`

