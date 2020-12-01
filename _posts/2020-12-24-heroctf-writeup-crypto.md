---
title: "Post: Modified Date"
last_modified_at: 2016-03-09T16:20:02-05:00
categories:

  - Blog

tags:

  - HeroCTF
  - Crypto
  - RSA

---

Last year I got to participate as a special guest in a CTF (Capture the Flag) organized by our university's student office. I scored 6th out of 45~ participants *and I'm still pissed that python2 and python3 don't encode characters the same way otherwhise I'd have solved that damn buffer overflow*. I've digressed slightly. Since I'm also part of the student office the organizers took me in this year as a challenge maker and tester. Sadly I didn't have time to finish two really cool programming challenges I'd started working on but I still managed to publish two cryptography tasks. Here are the writeups for them.



## RSArdoche

> fr : RSArdoche le roi du Salt n'aime pas qu'on lui vole ses Stack.

> en : RSArdoche the Salt king doesn't like getting his Stack stolen.

![](/assets/images/rsardoche.png)

We're two files, one containing the following code and an `output.txt` file. The vulnerability sticks right out, `e = 1` is absolutely terrible practice. 

Let's recall the formula for RSA encryption : `ct = pt ^ e % n`

The strength of the RSA cryptosystem lies in the fact it is near impossible to factor large prime numbers. It won't be necessary here because for `e = 1` we have :

```mathematica
        ct = pt ^ e % n
    <=> ct = pt ^ 1 % n
    <=> ct = pt % n
    <=> ct = pt (since n > ct)
```

Since the ciphertext equal to the plaintext we can simply decode it.

```python
>>> from Crypto.Util.number import long_to_bytes
>>> print(long_to_bytes(ct).decode())
Hero{m41s_c5t4it_5ur_3nf41t_c5t41t_5ur!!!}
```

This challenge is based on a [real world example](https://www.cryptofails.com/post/70059600123/saltstack-rsa-e-d-1) where SaltStack used an exponent of 1 to generate RSA key pairs. This [stackoverflow post](https://stackoverflow.com/questions/17490282/why-is-this-commit-that-sets-the-rsa-public-exponent-to-1-problematic) explains in more detail the vulnerability in question.



## Postman

> fr : Le facteur passe parfois à l'eur. Reconstruisez une clé privée à partir de ces clés publiques et connectez vous avec `ssh -i KEYFILE postman@challs.heroctf.fr -p 22555`

> en : The postman sometimes arrives on time. Reconstruct his private key from these public keys and connect to his server with `ssh -i KEYFILE postman@challs.heroctf.fr -p 22555`
>
> "Facteur" in french means both postman and factor. "à l'eur" is phonetically similar to "Euler" 

We're given only one file, a `.tar.gz` archive containing a thousand public ssh keys. Yep, that's quite a few. By inspecting a few random keys we can see that they're all 4096 bit keys with an exponent of 65537. With such a large number of keys the first attack to test is a common factor attack, this [blogpost](http://www.loyalty.org/~schoen/rsa/) explains the attack in great detail. The jist of it is that if two of the keys have a common factor (due to a faulty PRNG for example) we can factor both of them trivially with the euclidian algorithm. In the following example let `Na` and `Nb` be `N`'s of two public keys vulnerable to a common factor attack. 

```python
p = GCD(Na, Nb)
q = Na // p

phi = (p - 1) * (q - 1)
d   = inverse(0x10001, phi)

key = RSA.construct((Na, 0x10001, d))
print(key.export_key('PEM').decode()) # We've just found the private key
```

This is an attack that [RsaCtfTool](https://github.com/Ganapati/RsaCtfTool) can perform but a script to automate computing the GCD of each pair of two keys is fast and straightforward to make.

```python
from Crypto.PublicKey import RSA
from Crypto.Util.number import inverse
import math, os
from itertools import combinations
from tqdm import tqdm

ns = {}
path = 'keys'

for k in os.listdir(path):
	with open(path + '/' + k, 'r') as file:
		pub = RSA.import_key(file.read())
	ns[k] = pub.n

for x, y in tqdm(combinations(ns, 2)):
	p = math.gcd(ns[x], ns[y])
	if p != 1:
		print(f'Key {x} and {y} have a common factor:{p}')
		break
		
e  = 0x10001

q   = ns[x] // p
phi = (p - 1) * (q - 1)
d   = inverse(e, phi)

key = RSA.construct((ns[x], e, d))
print(key.export_key('PEM').decode())
```

The script takes approximately 3mins to complete and when it's done we can log into the server and cat the flag : ``Hero{3ul3r_b3_pr0ud}`` .

The [paper that this challenge is inspired by](https://eprint.iacr.org/2012/064.pdf) and the article I linked to up above references were hinted at in a image hint that teams could chose to unlock.

![](/assets/images/hint-postman.jpg)

I also discovered around 15mins before the end of the CTF that [a similar challenge](https://ctftime.org/writeup/14021) had already been made for a previous CTF which pains me deeply but objectively mine is better so I'm fine with the knowledge that there won't be any completely original ideas left at some point.  
