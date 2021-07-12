---
---
layout: post
title: A XOR Challenge
author: Arul Kumaran
permalink: /a-xor-challenge.html
---

## Description
Haha! I encrypted a message containing the flag with a 12-byte XOR key. Let's see if you can crack it. Btw, the phrase "Your flag is" shows up in the plaintext, good luck!
Note: The plaintext containing the flag contains some non-printable ascii characters to throw off auto-xor solvers :wink:
Flag format: FLAG{some_string}


[challenge.zip](https://github.com/xxdydx/blog/files/6801980/challenge.zip)


(Credits to Sean for the challenge)

## Solution
A classic XOR challenge. The trick of this challenge is to make use of XOR's unique properties. Mainly the property ```A ⊕ A = 0```. 


Let's assume the plaintext containing the flag is A, while the key is B. If we XOR these two together, we get ```A ⊕ B = C```. 
Then, if we apply the property above, and try to XOR the ciphertext with a known part of a ciphertext, we can try to find the key.
Like this, ```C ⊕ A = (A ⊕ B) ⊕ A = B```.


Hence, let's apply this in this script to solve for the flag.


First, we XOR the known part of the plaintext with the ciphertext.

```python
from binascii import unhexlify
from regex import findall
with open('cipher', "rb") as f:
    ciphertext = unhexlify(f.read().strip())

key = "Your flag is"
key = bytes(key, 'ascii')
keyList = []
x = 0
data = ciphertext
while x < len(ciphertext) - 12:
    res = ""
    try:
        res = res + chr(data[x] ^ key[0])
        res = res + chr(data[x+1] ^ key[1])
        res = res + chr(data[x+2] ^ key[2])
        res = res + chr(data[x+3] ^ key[3])
        res = res + chr(data[x+4] ^ key[4])
        res = res + chr(data[x+5] ^ key[5])
        res = res + chr(data[x+6] ^ key[6])
        res = res + chr(data[x+7] ^ key[7])
        res = res + chr(data[x+8] ^ key[8])
        res = res + chr(data[x+9] ^ key[9])
        res = res + chr(data[x+10] ^ key[10])
        res = res + chr(data[x+11] ^ key[11])
        res = res[-(x % 12):] + res[:-(x % 12)] #to account for first x letters in the ciphertext

    except Exception as e:
        break
    keyList.append(res)
    x = x + 1
    
```

Then, we XOR the list of possible keys with the ciphertext. We can then use the power of Regex to single out the plaintext containing the flag.


```python
data = ciphertext
for k in keyList:
    #k = bytes(k, 'ascii')
    counter = 0
    res = ''
    while counter < len(data) - 12:
        res = res + chr(data[counter] ^ ord(k[0]))
        res = res + chr(data[counter+1] ^ ord(k[1]))
        res = res + chr(data[counter+2] ^ ord(k[2]))
        res = res + chr(data[counter+3] ^ ord(k[3]))
        res = res + chr(data[counter+4] ^ ord(k[4]))
        res = res + chr(data[counter+5] ^ ord(k[5]))
        res = res + chr(data[counter+6] ^ ord(k[6]))
        res = res + chr(data[counter+7] ^ ord(k[7]))
        res = res + chr(data[counter+8] ^ ord(k[8]))
        res = res + chr(data[counter+9] ^ ord(k[9]))
        res = res + chr(data[counter+10] ^ ord(k[10]))
        res = res + chr(data[counter+11] ^ ord(k[11]))
        counter = counter + 12
    if findall('FLAG{.+}', res):
        print(findall('FLAG{.+}', res))
        break
```

And then we get the flag, ```FLAG{gg_y0u_kn0w_h0w_t0_und0_x0r_w17h_k3y_l3ngth_4nd_par7ial_pl41n73x7_kn0wl3d63}```.


## Appendix
This is the challenge author's code. I didn't know that pwntools has such an amazing XOR function lol, you don't even need to convert the ciphertext to bytes.

```python
from binascii import unhexlify
from pwn import *
import regex as re


def pad_key(key, keylength):
    perm = i % keylength
    key_fixed = key[-perm:] + key[:-perm]
    return key_fixed


msgenc = open('seancipher', 'r').read()
msg = unhexlify(msgenc.strip())

part_of_msg = b"Your flag is"
partial_pt = b"FLAG{"
keylength = 12

with log.progress("enumerating") as pro:
    for i in range(len(msg)):
        candidate = xor(msg[i:i+len(part_of_msg)], part_of_msg)
        candidate_padded = pad_key(candidate, keylength)
        possible_pt = xor(msg, candidate_padded)
        pro.status(f"{i}")
        if (partial_pt in possible_pt):
            print(re.findall(b"FLAG{.+}", possible_pt).pop().decode())
            break
            
```


