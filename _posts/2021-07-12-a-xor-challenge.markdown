---
layout: post
title: A XOR Challenge
author: Arul Kumaran
permalink: /a-xor-challenge.html
---


## Description
Haha! I encrypted a message containing the flag with a 12-byte XOR key. Let's see if you can crack it. Btw, the phrase "Your flag is" shows up in the plaintext, good luck!
Note: The plaintext containing the flag contains some non-printable ascii characters to throw off auto-xor solvers 😉
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
        for i in range(len(key)):
            res += chr(data[x + i] ^ key[i % len(key)])

    except Exception as e:
        break
    res = res[-(x % 12):] + res[:-(x % 12)] #padding
    keyList.append(res)
    x = x + 1

    
```
*The padding function is used to account for the previous x letters of the ciphertext, and wraps-around the key according to the position of the character in each block of 12 characters.*


Then, we XOR the list of possible keys with the ciphertext. We can then use the power of Regex to single out the plaintext containing the flag.


```python
data = ciphertext
for k in keyList:
    #k = bytes(k, 'ascii')
    counter = 0
    res = ''
    for j in range (len(ciphertext)):
        res += chr(data[j] ^ ord(k[j % len (k)]))
    if findall('FLAG{.+}', res):
        print(findall('FLAG{.+}', res))
        break
```

And then we get the flag, FLAG{gg_y0u_kn0w_h0w_t0_und0_x0r_w17h_k3y_l3ngth_4nd_par7ial_pl41n73x7_kn0wl3d63}.


## Appendix
This is the challenge author's code. 
```python
#python 2 please
import string
import binascii

def xor(msg, key):
    o = ''
    for i in range(len(msg)):
        o += chr(ord(msg[i]) ^ ord(key[i % len(key)]))
    return o
  
def pad_key(key,keylength):
    key_fixed = ""
    list_key=[]
    list_key[:0]=key
    for j in range(i%keylength):
        tmp = list_key[-1]
        list_key.pop(-1)
        list_key.insert(0,tmp)
    key_fixed = ''.join(str(i) for i in list_key)
    return key_fixed


msg = binascii.unhexlify("")

part_of_msg = "Your flag is"
partial_pt = "FLAG{"
keylength = 12


for i in range(len(msg)):
    print i,
    candidate = xor(msg[i:i+len(part_of_msg)], part_of_msg)
    candidate_padded = pad_key(candidate,keylength)
    possible_pt = xor(msg,candidate_padded)
    if (partial_pt in possible_pt):
        print(possible_pt)
            
```

My full code:
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
        for i in range(len(key)):
            res += chr(data[x + i] ^ key[i % len(key)])

    except Exception as e:
        break
    res = res[-(x % 12):] + res[:-(x % 12)]
    keyList.append(res)
    x = x + 1

data = ciphertext
for k in keyList:
    #k = bytes(k, 'ascii')
    counter = 0
    res = ''
    for j in range (len(ciphertext)):
        res += chr(data[j] ^ ord(k[j % len (k)]))
    if findall('FLAG{.+}', res):
        print(findall('FLAG{.+}', res))
        break
```

