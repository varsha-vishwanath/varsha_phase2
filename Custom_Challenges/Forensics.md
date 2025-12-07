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



## RAR of the Abyss

### Description
Two philosophers peer into the networked abyss and swap a secret. Use the secret to decrypt the Abyss’ RAwR and pull your flag from the void.

### Solution:
I used  `wireshark` to analyze `abyss.pcap` and then filtered to find nonempty tcp packages. After looking through the packages I found a password and a  `.rar` file. Finally, I exported the rar file and used the password to unrar it which gave me the flag. 

<img width="1919" height="1134" alt="Screenshot 2025-12-07 212113" src="https://github.com/user-attachments/assets/670099ad-dc13-46e3-a7c8-b2c72cc6f097" />

<img width="645" height="160" alt="Screenshot 2025-12-07 212121" src="https://github.com/user-attachments/assets/e47cf8e3-6a8f-4928-9020-0fbb34dc2df2" />

<img width="1916" height="1132" alt="image" src="https://github.com/user-attachments/assets/a9dd0460-e4ae-4f8f-b053-34269ad8aa45" />

<img width="1279" height="410" alt="abyss" src="https://github.com/user-attachments/assets/b4542688-3071-4e0b-a65b-555f49f0b3f0" />

### Flag:
**Flag:** `nite{thus_sp0k3_th3_n3tw0rk_f0r3ns1cs_4n4lyst}`



## NineTails

### Description:
Looks like I got a little too clever and hid the flag as a password in Firefox, tucked away like one of NineTails’ many tails. Recover the "logins" and the "key4" and let it guide you to the flag.

Hint:
I named my Ninetails "j4gjesg4", quite a peculiar name isn't it?

### Solution:
I extracted the `.ad1` file from the given `.rar` and then used `FTK Imager` to look for the files: `logins.json` and `key4.db`. Then I used `firefox_decrypt` to get the passwords and putting those together got me the flag. Originally, I downloaded an FTK software that didn't work correctly so it took me a bit to find one that worked. The previous one kept returning a `no valid evidence` error.

<img width="1413" height="520" alt="image" src="https://github.com/user-attachments/assets/f6cd4872-0503-45e4-adcc-e32f082b271b" />

<img width="1628" height="738" alt="Screenshot 2025-12-07 184717" src="https://github.com/user-attachments/assets/5e5b88ba-5136-45cb-bb4a-8ffe8bd992b0" />

### Flag:
**Flag:** `GCTF{m0zarella_f1ref0x_p4ssw0rd}`



## Re:Draw

### Description:
Her screen went black and a strange command window flickered to life, lines of text flashed across before everything went silent. Moments later, the system crashed. By sheer luck, we recovered a memory dump. 

Note: There are three stages to this challenge and you will find three flags.

What we know: just before the crash, a black command window flickered across the screen, something in its output might still be visible if you dig through memory. She was drawing when it happened, and remnants of a painting program linger, which could reveal more if inspected in the right way. Finally, a mysterious archive hides deeper in memory, likely holding the last piece of her work.

Hint:
Learn up on volatility 2 and its various plugins and what they are used for.

### Solution
Instead of using wsl, I used powershell over here because it was easier to use `volatility 2` on it. Here's what I did:

1. For stage 1, I identified the profile and looked into the command history to find the flag Base64 encoded. I decoded that to find the first flag:

```
=== STAGE 1: Command Window Analysis ===
PS C:\Users\Varsha\Downloads> & $vol -f $dump --profile=$profile cmdscan
Volatility Foundation Volatility Framework 2.6
**************************************************
CommandProcess: conhost.exe Pid: 2692
CommandHistory: 0x1fe9c0 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 1 LastAdded: 0 LastDisplayed: 0
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x60
Cmd #0 @ 0x1de3c0: St4G3$1
Cmd #15 @ 0x1c0158:
Cmd #16 @ 0x1fdb30:
**************************************************
CommandProcess: conhost.exe Pid: 2260
CommandHistory: 0x38ea90 Application: DumpIt.exe Flags: Allocated
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x60
Cmd #15 @ 0x350158: 8
Cmd #16 @ 0x38dc00: 8
PS C:\Users\Varsha\Downloads> & $vol -f $dump --profile=$profile consoles
Volatility Foundation Volatility Framework 2.6
**************************************************
ConsoleProcess: conhost.exe Pid: 2692
Console: 0xff756200 CommandHistorySize: 50
HistoryBufferCount: 1 HistoryBufferMax: 4
OriginalTitle: %SystemRoot%\system32\cmd.exe
Title: C:\Windows\system32\cmd.exe - St4G3$1
AttachedProcess: cmd.exe Pid: 1984 Handle: 0x60
----
CommandHistory: 0x1fe9c0 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 1 LastAdded: 0 LastDisplayed: 0
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x60
Cmd #0 at 0x1de3c0: St4G3$1
----
Screen 0x1e0f70 X:80 Y:300
Dump:
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\SmartNet>St4G3$1
ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0=
Press any key to continue . . .
```

2. For stage 2, I found mspaint in the processes so i looked into the memory dump to find that `2424.dmp` was created. I opened this file as raw data in `gimp` and messed around with the dimensions till I found the flag:

```
PS C:\Users\Varsha\Downloads> & $vol -f $dump --profile=$profile pslist | Select-String -Pattern "paint|mspaint|draw" -CaseSensitive:$false
Volatility Foundation Volatility Framework 2.6

0xfffffa80022bab30 mspaint.exe            2424    604      6      128      1      0 2019-12-11 14:35:14 UTC+0000
```

<img width="1914" height="1127" alt="Screenshot 2025-12-07 193306" src="https://github.com/user-attachments/assets/b83c7c75-4463-4a5b-bfd9-754d69a3433c" />

3. For stage 3, I found `Important.rar` at `\Users\Alissa` and `Simpson\Documents\Important.rar`. I extracted the file and the NTLM hashes using `volatility --profile=Win7SP1x64 -f MemoryDump_Lab1.raw hashdump`. Then I used `7zip` to unzip the folder and open the flag using one of the hashes itself as the password. This stage took me a while as I originally used an online decrypter to decrypt the hashes instead.

```
PS C:\Users\Varsha\Downloads> & $vol -f $dump --profile=$profile filescan | Select-String -Pattern "\.(zip|rar|7z|tar)" -CaseSensitive:$false | Select -First 10
Volatility Foundation Volatility Framework 2.6

0x000000003fa3ebc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
0x000000003fac3bc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
0x000000003fb48bc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
```

<img width="1913" height="1142" alt="Screenshot 2025-12-07 193945" src="https://github.com/user-attachments/assets/7d4daf60-88a2-45c6-8d0e-39b207a39f22" />



