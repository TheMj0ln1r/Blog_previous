---
layout: post
title:  "m0leCon Beginner CTF 2022"
command: cat m0leCon
---

I played **m0lecon CTF 2022** with my CTF team [TEAM-ASGARD](https://ctftime.org/team/196057). These are the few challenges those i solved.


# Crypto - Unrecognizable

The challenge contains these two images.

![challenge](/assets/img/post_img/m0lecon_unrecognizable1.png){: width="200" height="200"}

![whoami](/assets/img/post_img/m0lecon_unrecognizable2.png){: width="200" height="200"}

Immediately i wrote this script to make `xor` of these two images.

```python
from PIL import Image, ImageChops
im1 = Image.open('challenge.png')
im2 = Image.open('whoami.png')

im3 = ImageChops.add(ImageChops.subtract(im2, im1), ImageChops.subtract(im1, im2))
im3.save("im3.png")
```
Which gives new image consist of the flag in it.

![flag](/assets/img/post_img/m0lecon_unrecognizable3.png){: width="200" height="200"}

> `Flag: ptm{y0u_r34lly_7r4c3d_m3}`

***

# Misc - f33linegCut3

The challenge contains this selfie.jpeg image.

![feelingCut3](/assets/img/post_img/m0lecon_selfie.jpeg){: width="300" height="300"}

Used `exiftool` to read metadata. `exiftool selfie.jpeg` and found this comment in metadata.

Comment:010101000110100001100101010011010110100101100111011010000111010001111001010100000110000101110111

Then I followed this `Bin > Hex > Ascii` Using <a href="https://gchq.github.io/CyberChef/" target=_blank>Cyberchef</a>. 

The result is of this is `TheMightyPaw`

Now I tried `steghide extract -sf selfie.jpeg` and it results a `mesage.zip` which is password protected.

Unzipped the `message.zip` using `TheMightyPaw` as the password. It gave us the flag.

> `Flag : ptm{n33d_M0r3_c4TN1p_b0ss}`

***

# Misc - FileRecover

In this challenge a `capture.pcap` is attached.

By analyzing it in wireshark. I found that in one packet large size of data has been transmitted. I exported the packet data as the Image. The image contains the flag we need.

![pcap](/assets/img/post_img/m0lecon_pcap){: width="300" height="300"}

> `Flag : n37w0rk_ch4ll3n935_4r3_fun_2`

***