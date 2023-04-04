---
layout: post
title:  "Cryptoverse CTF 2022"
command: cat cryptoverse
---

Hi friends!

I played another CTF named Cryptoverse CTF 2022.
Cryptoverse CTF 2022 is a beginner-friendly CTF organized by an individual who is particularly interested in Cryptography and Reverse Engineering.

Let me explain the challs i solved in Cryptoverse CTF 2022.

# Baby Reverse

This is a basic reverse challenge.

![baby_reverse](/assets/img/post_img/babyreverse1.png){: w="600" h="600"}

This can be solved by using strings command in linux

```bash
$ strings chall
```
> `Flag : cvctf{7h15_15_4_f4k3_fl4g}`

***

# Crypto/Warmup 1

![crypto/warmup1](/assets/img/post_img/cryptowarmup1.png){: w="600" h="600"}

The task is to decode the base64 cipher "cGlwZ3N7cG5yZm5lXzY0X3Nnan0=".

```bash
$ echo "cGlwZ3N7cG5yZm5lXzY0X3Nnan0=" | base64 -d
$ pipgs{pnrfne_64_sgj}
```
Then do a rot13 on the deciphered text.
```bash
$ echo "pipgs{pnrfne_64_sgj}" | rot13
$ cvctf{caesar_64_ftw}
```
> `Flag : cvctf{caesar_64_ftw}`

***

# Crypto/Warmup 2

![crypto/warmup2](/assets/img/post_img/cryptowarmup2.png){: w="600" h="600"}

The cipher is fzvxw{hqtegmfr_lw_msf_scrslg_kvwlhyk_fpr_kxg?} which is a vegener cipher and the key to decrypt the cipher is determination. 
I used [this](https://www.dcode.fr/vigenere-cipher) site to do it.

> `Flag : cvctf{vigenere_is_too_guessy_without_the_key?}`

***

# Crypto/Warmup 3

![crypto/warmup3](/assets/img/post_img/cryptowarmup3.png){: w="600" h="600"}

-.-. ...- -.-. - ..-. -- ----- .-. ..... ...-- .. ... -. ----- - ..... ----- ..-. ..- -. is the morse code. I used [CyberChef](https://gchq.github.io/CyberChef/) to convert morse code to plaintext.

> `Flag : cvctf{m0r53isn0t50fun}`
***

# Crypto/Substitution

![crypto/Substitution](/assets/img/post_img/cryptosubstitution.png){: w="600" h="600"}

I used [This](https://www.guballa.de/substitution-solver) website to solve this.

> `Flag : cvctf{averysimplesubstitution}`
***

# Crypto/RSA 1

![crypto/RSA1](/assets/img/post_img/cryptorsa1.png){: w="600" h="600"}

This is a RSA challenge with small e. And i used factordb to find the primes of N, then i computed the phi to find the d which can be used to decrypt the cipher.

```python
from utilis import egcd,modinv,Convert
from Factorizations.factordb import *
try:
    def factordb(n):
        f =  FactorDB(n)
        f.connect()
        return f.get_factor_list()
    n = "0x7c05a45d02649367ebf6f472663119777ce5f9b3f2283c7b03471e9feb1714a3ce9fa31460eebd9cd5aca7620ecdb52693a736e2fcc83d7909130c6038813fd16ef50c5ca6f491b4a8571289e6ef710536c4615604f8e7aeea606d4b5f59d7adbec935df23dc2bbc2adebbee07c05beb7fa68065805d8c8f0e86b5c3f654e651"
    e = "0x10001"
    c = "0x35b63f7513dbb828800a6bcd708d87a6c9f33af634b8006d7a94b7e3ba62e6b9a1732a58dc35a8df9f7554e1168bfe3de1cb64792332fc8e5c9d5db1e49e86deb650ee0313aae53b227c75e40779a150ddb521f3c80f139e26b2a8880f0869f755965346cd28b7ddb132cf8d8dcc31c6b1befc83e21d8c452bcce8b9207ab76e"
    n = int(n,16)
    e = int(e,16)
    c = int(c,16)
    factordb = factordb(n)
    q = factordb[0]
    p = factordb[1]
    phi = (p-1)*(q-1)
    d = modinv(e,phi)
    decode = pow(c,d,n)
    Convert(decode)
except IndexError:
    print("[-] Sorry Can't Factorize n ")
```
> `Flag : cvctf{f4c70rDB_15_p0w3rfu1}`

***