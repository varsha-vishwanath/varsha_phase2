# 1. RSA Oracle

Can you abuse the oracle?
An attacker was able to intercept communications between a bank and a fintech company. They managed to get the message (ciphertext) and the password that was used to encrypt the message.
After some intensive reconassainance they found out that the bank has an oracle that was used to encrypt the password and can be found here nc titan.picoctf.net 59094. Decrypt the password and use it to decrypt the message. The oracle can decrypt anything except the password.

## Solution

- I read up on RSA oracle and ran this python code using output from the nc session in order to solve this challenge:
```
from subprocess import run, PIPE  
  
c = 2575135950983117315234568522857995277662113128076071837763492069763989760018604733813265929772245292223046288098298720343542517375538185662305577375746934
    
print(f"c = {c}\n")     
 
m1 = input("Enter message (m1): ")  
m1_bytes = bytes(m1, "utf-8")  
m1_int = ord(m1_bytes) 
  
print(f"Have the oracle encrypt this message (m1): {m1}\n")  
c1 = int(input("Enter ciphertext from oracle (c1 = E(m1)): "))  
print("\n")  
 
c2 = c * c1  
print(f"Have the oracle decrypt this message (c2 = c * c1): {c2}\n")  
  
m2 = int(input("Enter decrypted ciphertext as HEX (m2 = D(c2): "), 16)  
print("\n")  
 
m_int = m2 // m1_int  
m = m_int.to_bytes(len(str(m_int)), "big").decode("utf-8").lstrip("\x00")
print(f"Password (m = m2 / m1): {m}\n")  
  
print("-" * 50)  
   
res = run([r"C:\Program Files\OpenSSL-Win64\bin\openssl.exe",
           "enc", "-aes-256-cbc", "-d", "-in", "secret.enc", "-pass",
           f"pass:{m}"], stdout=PIPE, stderr=PIPE, text=True)

print(res.stdout)
```
- Here's the output:
```
c = 2575135950983117315234568522857995277662113128076071837763492069763989760018604733813265929772245292223046288098298720343542517375538185662305577375746934

Enter message (m1): a
Have the oracle encrypt this message (m1): a

Enter ciphertext from oracle (c1 = E(m1)): 1894792376935242028465556366618011019548511575881945413668351305441716829547731248120542989065588556431978903597240454296152579184569578379625520200356186


Have the oracle decrypt this message (c2 = c * c1): 4879347969494695763992079257309920962956235340071608197771324952441750870022564003875958517877187374665821306399293178559399723408243829615035132429286786526803773011124941819240588649086684776588479763949051499637834997128787647253068836995761445652838138074231225267441000841604830001446657688224597433724

Enter decrypted ciphertext as HEX (m2 = D(c2): 1305d947a822


Password (m = m2 / m1): 24bcb

--------------------------------------------------
picoCTF{su((3ss_(r@ck1ng_r3@_24bcbc66}
```

## Flag:

`picoCTF{su((3ss_(r@ck1ng_r3@_24bcbc66}`

## Concepts Learned

- I learned about RSA and that it is a public-key cryptosystem for secure data transmission.
- It has the property `D(c1 * c2 mod n) = D(c1) * D(c2) mod n`

## Notes

I tried directly decrpyting the password with the oracle, but obviously that didn't work.


## Resources

- https://crypto.stackexchange.com/questions/3555/homomorphic-cryptosystems-in-rsa



## 2. Custom encryption

Can you get sense of this code file and write the function that will decode the given encrypted file content.
Find the encrypted file here flag_info and code file might be good to analyze and get the flag.

## Solution

- After looking at the code, I understood that it had a shared lock, a multiplication lock, and an XOR + reversal lock. I had to get the shared key and undo the rest.

- I used this python code in order to solve this challenge:

```
from custom_encryption import is_prime, generator
 
 
def leak_shared_key(a, b):
    p = 97
    g = 31
    if not is_prime(p) and not is_prime(g):
        print("Enter prime numbers")
        return
    u = generator(g, a, p)
    v = generator(g, b, p)
    key = generator(v, a, p)
    b_key = generator(u, b, p)
    shared_key = None
    if key == b_key:
        shared_key = key
    else:
        print("Invalid key")
        return
 
    return shared_key
 
 
def decrypt(ciphertext, key):
    semi_ciphertext = []
    for num in ciphertext:
        semi_ciphertext.append(chr(round(num / (key * 311))))
    return "".join(semi_ciphertext)
 
 
def dynamic_xor_decrypt(semi_ciphertext, text_key):
    plaintext = ""
    key_length = len(text_key)
    for i, char in enumerate(semi_ciphertext):
        key_char = text_key[i % key_length]
        decrypted_char = chr(ord(char) ^ ord(key_char))
        plaintext += decrypted_char
    return plaintext[::-1]
 
 
if __name__ == "__main__":
    # 0. Take relevant values from `enc_flag` and `custom_encryption.py`
    a = 94
    b = 29
    ciphertext_arr = [260307, 491691, 491691, 2487378, 2516301, 0, 1966764, 1879995, 1995687, 1214766, 0, 2400609, 607383, 144615, 1966764, 0, 636306, 2487378, 28923, 1793226, 694152, 780921, 173538, 173538, 491691, 173538, 751998, 1475073, 925536, 1417227, 751998, 202461, 347076, 491691]

    text_key = "trudeau"
     shared_key = leak_shared_key(a, b)
 
    semi_ciphertext = decrypt(ciphertext_arr, shared_key)
 
    plaintext = dynamic_xor_decrypt(semi_ciphertext, text_key)
 
    print(plaintext)
```

After looking at the code, I understood that it had a shared lock, a multiplication lock, and an XOR + reversal lock. 

## Flag:

`picoCTF{custom_d2cr0pt6d_751a22dc}`

## Concepts Learned

- Diffie–Hellman “shared lock” (twice): the code uses DH-style operations (public g, p and private exponents a, b) to produce a common numeric key. Both sides compute the same secret value without directly sharing their private numbers — that secret becomes the numeric key used later.

- Multiplicative obfuscation: each plaintext character is converted to its ordinal and multiplied by the shared numeric key and a fixed multiplier (311). This turns characters into large integers, hiding the original bytes by scaling.

- XOR with a repeating text key + reversal: the scaled characters are converted back into characters, XORed against a repeating text key (e.g., "trudeau"), and the result is reversed. XOR is symmetric (doing it twice restores the original), and reversing is trivially inverted.

## Resources

- https://www.geeksforgeeks.org/computer-networks/implementation-diffie-hellman-algorithm/
- https://www.geeksforgeeks.org/dsa/xor-cipher/



# 3. miniRSA

Let's decrypt this: ciphertext? Something seems a bit small.

## Solution

- I used a website to solve the problem:
  <img width="1176" height="677" alt="image" src="https://github.com/user-attachments/assets/670e2c81-5199-44e6-845f-c153e5841d7b" />

- In RSA you normally see c = m^e mod N. That mod N matters when m^e is as big as or bigger than N (it wraps around). But in this challenge, e and c are small compared to N. So we can treat c as the actual cube of the message integer m. That means the decryption is simply taking the integer cube root of c.

## Flag:

`picoCTF{n33d_a_lArg3r_e_606ce004}`

## Resources:

- https://www.dcode.fr/rsa-cipher
- https://crypto.stackexchange.com/questions/33561/cube-root-attack-rsa-with-low-exponent







# 2. Custom encryption
