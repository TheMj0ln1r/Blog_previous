---
layout: post
title:  "NullCon HackIM CTF 2023"
command: cat NullCon
tags: CTF
---

Long time no see? I have been learning things in Blockchain technology in last few months. Now everything at good pace hope I can solve more CTF's now. As comeback match, I solved this crypto challenge in <a href= "https://ctftime.org/event/2065" target=_blank>NullCon HackIM CTF 2023</a> yesterday.


# Crypto - EuclideanRSA

```text
Attached Files : [euclidean-RSA.py, output.txt]
```

`euclidean-RSA.py`

```python
#!/usr/bin/env python3
from Crypto.PublicKey import RSA
from Crypto.Util.number import bytes_to_long
from secret import flag, magic

while True:
	try:
		key = RSA.generate(2048)
		a,b,c,d = magic(key)
		break
	except:
		pass
assert a**2 + b**2 == key.n
assert c**2 + d**2 == key.n
for _ in [a,b,c,d]:
	print(_)
cipher = pow(bytes_to_long(flag), key.e, key.n)
print(cipher)

```

`output.txt`

```text
139488614271687589953884690592970545345100917058745264617112217132329766542251923634602298183777415221556922931467521901793230800271771036880075840122128322419937786441619850848876309600263298041438727684373621920233326334840988865922273325440799379584418142760239470239003470212399313033715405566836809419407
68334789534409058399709747444525414762334123566273125910569662060699644186162637240997793681284151882169786866201685085241431171760907057806355318216602175990235605823755224694383202043476486594392938995563562039702918509120988489287149220217082428193897933957628562633459049042920472531693730366503272507672
124011822519139836919119491309637022805378274989854408578991029026928119002489232335977596528581855016599610031796540079373031282148998625318658034408770283112750172223328012238338715644743140990997114236125750813379366262418292349962679006556523851370694404238101553966330965676189731474108393418372330606063
93529593432394381438671783119087013080855868893236377597950059020717371498802208966524066540234253992421963224344343067174201693672350438201011499762718490958025617655722916641293034417795512315497126128726644064013248230211347407788186320320456853993252621916838570027019607155835349111757703654306736031792
2819638499688340337879314536945338371611392232636746056275506290935131270682791584873534866138393305591899169164183372576878693678187335219904407119253951099126339949954303448641761723704171837075206394491403411400326176280981393624784437102905397888236098861970020785226848615566768625581096019917060387964269283048823007992992874533775547300443032304973521568046956516203101626941042560505073773998143068621715480774707735064134961852206070850277695448391038882766344567740211926618750074636868149063283746597347807257171871016202588384726430246523650462866812935130465049824665395626882280287488078029119879891722

```

Observations : 
1. The modulus(n) is written as sum of two squares in two different ways
2. a,b,c,d values and ciphertext is given
3. `e` is not given, probably `65537`

After searching about `sum of squares as modulus in RSA` I found this research paper https://www.mdpi.com/2297-8747/24/2/62


"""Consider a semi-prime number whose construction consists of primes $$ p1,p2, N = p1p2$$ , being Pythagorean,and having a representation on the Cartesian plane such that, $$p1 = x^2 + y^2$$. It can easily be shown that the product of two such primes can be represented as the sum of four squares from which two sums of two squares can be derived. For such a semi-prime, if the original construction is unknown and the sum of four squares is known, the original construction $$p1,p2$$ can be found."""

Lets look at `Eulers Factorization`

- **A semi-prime whose prime factors are Pythagorean can be expressed as the sum of four squares, from which two sums of squares can be derived.**

A semi-prime $$N = p1p2, p1 = a^2+b^2, p2 = c^2+d^2$$ is expressed as the sum of four squares, such that:

$$
N=p1p2=(a^2+b^2)(c^2+d^2)=(ac)^2+(bc)^2+(ad)^2+(bd)^2
$$

Now in our case, $$ N = (a^2+b^2)= (c^2+d^2)$$. We can solve for $$p1, p2$$ as follows.

$$
(a^2+b^2)= (c^2+d^2)
$$

combining odds we get,

$$
(a^2-c^2)= (d^2-b^2) \\
(a-c)(a+c) = (d-b)(d+b) \\

p1 = \left(\frac{gcd(a-c,d-b)}{2}\right)^2 + \left(\frac{gcd(a+c,d+b)}{2}\right)^2\\
p2 = \left(\frac{gcd(a+c,d-b)}{2}\right)^2 + \left(\frac{gcd(a-c,d+b)}{2}\right)^2\\
$$

Enough math, lets implement this simple math in python

`solve.py`

```python
from Crypto.Util.number import long_to_bytes
from sympy import *
import math
a = 139488614271687589953884690592970545345100917058745264617112217132329766542251923634602298183777415221556922931467521901793230800271771036880075840122128322419937786441619850848876309600263298041438727684373621920233326334840988865922273325440799379584418142760239470239003470212399313033715405566836809419407
b = 68334789534409058399709747444525414762334123566273125910569662060699644186162637240997793681284151882169786866201685085241431171760907057806355318216602175990235605823755224694383202043476486594392938995563562039702918509120988489287149220217082428193897933957628562633459049042920472531693730366503272507672
c = 124011822519139836919119491309637022805378274989854408578991029026928119002489232335977596528581855016599610031796540079373031282148998625318658034408770283112750172223328012238338715644743140990997114236125750813379366262418292349962679006556523851370694404238101553966330965676189731474108393418372330606063
d = 93529593432394381438671783119087013080855868893236377597950059020717371498802208966524066540234253992421963224344343067174201693672350438201011499762718490958025617655722916641293034417795512315497126128726644064013248230211347407788186320320456853993252621916838570027019607155835349111757703654306736031792
cipher =2819638499688340337879314536945338371611392232636746056275506290935131270682791584873534866138393305591899169164183372576878693678187335219904407119253951099126339949954303448641761723704171837075206394491403411400326176280981393624784437102905397888236098861970020785226848615566768625581096019917060387964269283048823007992992874533775547300443032304973521568046956516203101626941042560505073773998143068621715480774707735064134961852206070850277695448391038882766344567740211926618750074636868149063283746597347807257171871016202588384726430246523650462866812935130465049824665395626882280287488078029119879891722

n = (pow(a,2)+pow(b,2))

g1 = gcd(a-c, d-b)/2 
g2 = gcd(a+c, d+b)/2 

p = g1*g1 + g2*g2

g1 = gcd(a+c, d-b)/2 
g2 = gcd(a-c, d+b)/2 

q = g1*g1 + g2*g2

phi = (p-1)*(q-1)
print(phi)
d = pow(65537,-1,int(phi))
plaintext = long_to_bytes(pow(cipher, d, n))

print("Decrypted plaintext:", plaintext)

# Output
# Decrypted plaintext: b'ENO{Gauss_t0ld_u5_th3r3_1s_mor3_th4n_on3_d1men5i0n}'
```


# Crypto - Sebastian's Secret Sharing

```text
Attached files : [sss.py]
```
```python
#!/usr/bin/env python3
import random
from decimal import Decimal,getcontext

class SeSeSe:
	def __init__(self, s, n, t):
		self.s = int.from_bytes(s.encode(),byteorder="big")
		self.l = len(s) 	
		self.n = n
		self.t = t
		self.a = self._a()

	def _a(self):
		c = [self.s]
		for i in range(self.t-1):
			a = Decimal(random.randint(self.s+1, self.s*2))
			c.append(a)
		return c

	def encode(self):
		s = []
		for j in range(self.n):
			x = j
			px = sum([self.a[i] * x**i for i in range(self.t)]) 
			s.append((x,px))
		return s

	def decode(self, shares):
		assert len(shares)==self.t
		secret = Decimal(0)
		for j in range(self.t):
			yj = Decimal(shares[j][1])
			r = Decimal(1)
			for m in range(self.t):
				if m == j:
					continue
				xm = Decimal(shares[m][0])
				xj = Decimal(shares[j][0])

				r *= Decimal(xm/Decimal(xm-xj))
			secret += Decimal(yj * r)
		return int(round(Decimal(secret),0)).to_bytes(self.l).decode()


if __name__ == "__main__":
	getcontext().prec = 256 # beat devision with precision :D 
	n = random.randint(50,150)
	t = random.randint(5,10)
	sss = SeSeSe(s=open("flag.txt",'r').read(), n=n, t=t)
	
	shares = sss.encode()

	print(f"Welcome to Sebastian's Secret Sharing!")
	print(f"I have split my secret into 1..N={sss.n} shares, and you need t={sss.t} shares to recover it.")
	print(f"However, I will only give you {sss.t-1} shares :P")
	for i in range(1,sss.t):
		try:
			sid = int(input(f"{i}.: Choose a share: "))
			if 1 <= sid <= sss.n:
				print(shares[sid % sss.n])
		except:
			pass
	print("Good luck!")
```

The remote connection of chalenge gives us the following information.

```sh
mj0ln1r@Linux:~/ nc 52.59.124.14 10031
Welcome to Sebastian's Secret Sharing!
I have split my secret into 1..N=83 shares, and you need t=10 shares to recover it.
However, I will only give you 9 shares :P
1.: Choose a share: 
```

I searched about secret sharing schemes available, and found that this challenge is related to <a href="https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing" target=_blank>Shamir Secret Sharing</a>. I used trailofbits <a href="https://www.zkdocs.com/docs/zkdocs/protocol-primitives/shamir/" target=_blank>ZKdocs</a>to learn weaknesses in Shamir Secret Sharing.

### Shamir Secret Sharing

Shamir's Secret Sharing is a cryptographic algorithm that allows you to split a secret into multiple shares in such a way that a minimum threshold of these shares is required to reconstruct the original secret. The algorithm was developed by Adi Shamir in 1979.

#### Step 1: Choose a Prime Number and Parameters

1. Choose a large prime number $$p$$ such that $$p > \text{max_secret_value}$$, where $$\text{max_secret_value}$$ is the maximum possible value of the secret you want to share.
2. Choose a threshold value $$ (k) $$, which represents the minimum number of shares required to reconstruct the secret.
3. Choose $$(k-1)$$ random coefficients $$(a_1, a_2, \ldots, a_{k-1})$$ from the finite field $$(\mathbb{F}_p)$$.

#### Step 2: Define the Polynomial

4. Construct a polynomial of degree $$(k-1)$$ over $$(\mathbb{F}_p)$$ using the chosen coefficients:
   
   $$
   f(x) = a_1x^{k-1} + a_2x^{k-2} + \ldots + a_{k-1}x + \text{secret}\\
   $$

   Where:
   - $$(a_1, a_2, \ldots, a_{k-1})$$ are the randomly chosen coefficients.
   - $$(\text{secret})$$ is the value you want to share.

#### Step 3: Generate Shares

5. Choose $$(k)$$ distinct x-values $$(x_1, x_2, \ldots, x_k)$$ from the field $$(\mathbb{F}_p)$$.
6. Compute $$(y_i = f(x_i))$$ for each $$(i)$$, which represents the shares:

   $$
   y_i = a_1x_i^{k-1} + a_2x_i^{k-2} + \ldots + a_{k-1}x_i + \text{secret}
   $$

   These $$k$$ pairs $$((x_i, y_i))$$ are the shares that can be distributed among participants.

#### Step 4: Share Distribution

7. Distribute the shares $$((x_i, y_i))$$ to the participants. Each participant receives one share.

#### Step 5: Secret Reconstruction

8. To reconstruct the secret, you need at least $$(k)$$ shares. Let's say you have $$(k)$$ shares $$((x_1, y_1), (x_2, y_2), \ldots, (x_k, y_k))$$

9. Use Lagrange interpolation to find the value of the secret:

   $$
   \text{secret} = \sum_{i=1}^{k} y_i \cdot \frac{\prod_{j=1, j\neq i}^{k} (x - x_j)}{\prod_{j=1, j\neq i}^{k} (x_i - x_j)}
   $$

   Where:
   - $$(y_i)$$ are the y-values of the shares.
   - $$(x_i)$$ are the x-values of the shares.
   - $$(x)$$ is the variable (typically set to 0).

By calculating this equation, you can reconstruct the original secret.

This is how `Shamir Secret Sharing` cryptosystem works. Coming to Challenge, we are able to get the $$(k-1)$$ shares of secret, but we need exactly $$k$$ shares to reconstruct the secret. The secret is nothing but the $$0^{th}$$ share ((0,S_0)).

I was not able to find the flaw in system during the CTF, after the CTF **`hakid29`** on discord helped me to find the flaw, that is, sending `n` will give us the $$0^{th}$$ share which is the actual secret. 

**Why sending n..?**

If we observer the following line in code,

```python
sid = int(input(f"{i}.: Choose a share: "))
if 1 <= sid <= sss.n:
	print(shares[sid % sss.n])
```

we can know that this allows us to input the any number from (1 to n) including both. So in the below line simple $$%$$ operator is used to select the share among all shares. If we send `n` then it will become $$n mod n = 0$$, so it will select the $$0^{th}$$ share which is the flag we need.

```sh
mj0ln1r@Linux:~$ nc 52.59.124.14 10031
Welcome to Sebastian's Secret Sharing!
I have split my secret into 1..N=141 shares, and you need t=5 shares to recover it.
However, I will only give you 4 shares :P
1.: Choose a share: 141
(0, Decimal('111370287875855598506538509804271500535681803123044982950094717'))
2.: Choose a share: 
^C
```

```python
from Crypto.Util.number import *
print(long_to_bytes(111370287875855598506538509804271500535681803123044982950094717))

# output
# b'ENO{SeCr3t_Sh4m1r_H4sh1ng}
```

> I solved an easy web challenge too, look at this <a href="https://xhacka.github.io/posts/writeup/2023/08/20/TYPicalBoss/" target=_blank>blog</a> someone explained it better. 

***

Thank you for reading!