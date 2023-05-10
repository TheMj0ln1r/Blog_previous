---
layout: post
title:  "CryptoverseCTF 2023"
command: cat CryptoverseCTF
---

Hello all! 
I played <a href= "https://ctftime.org/event/1907" target=_blank>CryptoverseCTF 2023</a> which was happened from  06 May to 08 May. I played it with my team [Invaders0x1](https://ctftime.org/team/217079). This time a couple of automation tasks.

These are the writeups for the challenges I solved.

# Crypto

# Warmup 3

![warmup2](/assets/img/post_img/cvctf23_crypto2.png)

The Cipher given was a `Bifid Cipher` (from wikipedia). So, simply decoded it using <a href="https://www.dcode.fr/bifid-cipher" target=_blank> this site </a>

> `Flag : cvctf{funbiffiddecoding}}`

***

# Forensics

# The Cyber Heist

![forensic](/assets/img/post_img/cvctf23_cyberh.png)

Attached File : <a href="/assets/files/cvctf23/challenge.pcapng">challenge.pcapng</a>

There is only one forensic chall and it was fun to do. I opened the pcapng file in wireshark and it there is only the USB data traffic is recorded in the pcapng.

![forensic](/assets/img/post_img/cvctf23_cyberh1.png)


So, I did a quick search on google about USB traffic. Found <a href="https://ctf-wiki.mahaloz.re/misc/traffic/protocols/USB/" target=_blank>this blog </a>. So, I got to know that, I have to collect the keystrokes of the keyboard and decode it from the traffic to the printable characters.

First of all, I extracted the keystroke data field from the pcapng. The keyboard data is represented in `8 bytes` and the field of this 8 bytes data is `usbhid.data` 

![forensic](/assets/img/post_img/cvctf23_cyberh2.png)

So, I used `Tshark` to extract the data in this field.

```sh
mj0ln1r@Linux:~/foresics$ tshark -r challenge.pcapng -T fields -e usbhid.data -Y "usb.data_len == 8" > usbdata.txt
mj0ln1r@Linux:~/foresics$ head usbdata.txt
00ffff00ffffffff
00fefe00fefffeff
00feff00feffffff
00feff00feffffff
00fdfd00fdfffdff
00fdff00fdffffff
00fdfd00fdfffdff
00fdfe00fdfffeff
```

So, The next step is to convert these bytes to ASCII. And here is my solution script to solve this.

```python
#!/usr/bin/env python
import sys
import os
DataFileName = "usbdata.txt"
presses = []
normalKeys = {"04":"a", "05":"b", "06":"c", "07":"d", "08":"e", "09":"f", "0a":"g", "0b":"h", "0c":"i", "0d":"j", "0e":"k", "0f":"l", "10":"m", "11":"n", "12":"o", "13":"p", "14":"q", "15":"r", "16":"s", "17":"t", "18":"u", "19":"v", "1a":"w", "1b":"x", "1c":"y", "1d":"z","1e":"1", "1f":"2", "20":"3", "21":"4", "22":"5", "23":"6","24":"7","25":"8","26":"9","27":"0","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":"<SPACE>","2d":"-","2e":"=","2f":"[","30":"]","31":"\\","32":"<NON>","33":";","34":"'","35":"<GA>","36":",","37":".","38":"/","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}
shiftKeys = {"04":"A", "05":"B", "06":"C", "07":"D", "08":"E", "09":"F", "0a":"G", "0b":"H", "0c":"I", "0d":"J", "0e":"K", "0f":"L", "10":"M", "11":"N", "12":"O", "13":"P", "14":"Q", "15":"R", "16":"S", "17":"T", "18":"U", "19":"V", "1a":"W", "1b":"X", "1c":"Y", "1d":"Z","1e":"!", "1f":"@", "20":"#", "21":"$", "22":"%", "23":"^","24":"&","25":"*","26":"(","27":")","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":"<SPACE>","2d":"_","2e":"+","2f":"{","30":"}","31":"|","32":"<NON>","33":":","34":"\"","35":"<GA>","36":"<","37":">","38":"?","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}
def main():
    # check argv
    if len(sys.argv) != 2:
        print("Usage : ")
        print("        python newsolve.py challenge.pcapng")
        exit(1)
    # get argv
    pcapFilePath = sys.argv[1]
    # get data of pcap
    os.system("tshark -r %s -T fields -e usbhid.data -Y 'usb.data_len == 8' > %s" % (pcapFilePath, DataFileName))
    with open(DataFileName, "r") as f:
        for line in f:
            presses.append(line[0:-1])
    result = ""
    for press in presses:
        if press == '':
            continue
        if ':' in press:
            Bytes = press.split(":")
        else:
            Bytes = [press[i:i+2] for i in range(0, len(press), 2)]
        if Bytes[0] == "00":
            if Bytes[2] != "00" and normalKeys.get(Bytes[2]):
                result += normalKeys[Bytes[2]]
        elif int(Bytes[0],16) & 0b10 or int(Bytes[0],16) & 0b100000: # shift key is pressed.
            if Bytes[2] != "00" and normalKeys.get(Bytes[2]):
                result += shiftKeys[Bytes[2]]
        else:
            print("[-] Unknow Key : %s" % (Bytes[0]))
    print("[+] Found : %s" % (result))
if __name__ == "__main__":
    main()

#cvctf{w1r3shark_fun!!!}
```

> `Flag : cvctf{w1r3shark_fun!!!}`

***

# Rev

# Simple Checkin

![simple checkin](/assets/img/post_img/cvctf23_checkin.png)

Attached File : <a href="/assets/files/cvctf23/challenge">challenge</a>

A binary file was given, I used <a>ghidra</a> to view the source code of it. 

![simple checkin](/assets/img/post_img/cvctf23_checkin1.png)

In main function I observed that there were two long array are assigned to some values. And when I scroll down an `XOR` operation was done between these two array elements.

```C
  for (local_78 = 0; local_78 < local_7c; local_78 = local_78 + 1) {
		    if ((local_68[local_78] ^ (int)(char)local_48[local_78]) != local_58[local_78]) {
		      local_74 = 0;
		    }
	}
```

So, I copied the values of those two arrays performed a simple XOR between them, it resulted the flag.

Here, is the solution script

```python
local_68 = [i for i in range(0xe1+1)]
local_58 = [i for i in range(0xe1+1)]

local_68[0] = 0xb
local_58[0] = 0x68
local_68[1] = 0x7d
local_58[1] = 0xb
local_68[2] = 0xac
local_58[2] = 0xcf
local_68[3] = 0xa8
local_58[3] = 0xdc
local_68[4] = 5
local_58[4] = 99
local_68[5] = 0xef
local_58[5] = 0x94
local_68[6] = 0x1e
local_58[6] = 0x77
local_68[7] = 0xd7
local_58[7] = 0x88
local_68[8] = 0x8c
local_58[8] = 0xed
local_68[9] = 0x94
local_58[9] = 0xe4
local_68[10] = 0x1f
local_58[10] = 0x70
local_68[0xb] = 0xeb
local_58[0xb] = 0x87
local_68[0xc] = 0x9b
local_58[0xc] = 0xf4
local_68[0xd] = 0x2a
local_58[0xd] = 0x4d
local_68[0xe] = 0xb6
local_58[0xe] = 0xdf
local_68[0xf] = 0x80
local_58[0xf] = 0xfa
local_68[0x10] = 0x3a
local_58[0x10] = 0x5f
local_68[0x11] = 0x39
local_58[0x11] = 0x66
local_68[0x12] = 0x39
local_58[0x12] = 0x5f
local_68[0x13] = 0x5f
local_58[0x13] = 0x30
local_68[0x14] = 0x93
local_58[0x14] = 0xe1
local_68[0x15] = 0x5f
local_58[0x15] = 0
local_68[0x16] = 0x70
local_58[0x16] = 3
local_68[0x17] = 0x46
local_58[0x17] = 0x33
local_68[0x18] = 0xe1
local_58[0x18] = 0x82
local_68[0x19] = 0xb9
local_58[0x19] = 0xd1
local_68[0x1a] = 0x3c
local_58[0x1a] = 99
local_68[0x1b] = 0x3b
local_58[0x1b] = 0x5a
local_68[0x1c] = 0x76
local_58[0x1c] = 0x29
local_68[0x1d] = 0x2c
local_58[0x1d] = 0x40
local_68[0x1e] = 0x48
local_58[0x1e] = 0x27
local_68[0x1f] = 0xae
local_58[0x1f] = 0xc0
local_68[0x20] = 0x6a
local_58[0x20] = 0xd
local_68[0x21] = 0xd7
local_58[0x21] = 0x88
local_68[0x22] = 0x4b
local_58[0x22] = 0x38
local_68[0x23] = 0xa2
local_58[0x23] = 0xd6
local_68[0x24] = 0xbf
local_58[0x24] = 0xcd
local_68[0x25] = 0xc1
local_58[0x25] = 0xa8
local_68[0x26] = 0xee
local_58[0x26] = 0x80
local_68[0x27] = 0xba
local_58[0x27] = 0xdd
local_68[0x28] = 100
local_58[0x28] = 0x3b
local_68[0x29] = 0xbb
local_58[0x29] = 0xd2
local_68[0x2a] = 0xbb
local_58[0x2a] = 0xd5
local_68[0x2b] = 0xb7
local_58[0x2b] = 0xe8
local_68[0x2c] = 0xb6
local_58[0x2c] = 0xc2
local_68[0x2d] = 0xc6
local_58[0x2d] = 0xae
local_68[0x2e] = 0x49
local_58[0x2e] = 0x20
local_68[0x2f] = 0xe0
local_58[0x2f] = 0x93
local_68[0x30] = 0xb3
local_58[0x30] = 0xec
local_68[0x31] = 0x23
local_58[0x31] = 0x40
local_68[0x32] = 0x85
local_58[0x32] = 0xed
local_68[0x33] = 0x58
local_58[0x33] = 0x3d
local_68[0x34] = 0xa1
local_58[0x34] = 0xc2
local_68[0x35] = 0xd
local_58[0x35] = 0x66
local_68[0x36] = 0xf3
local_58[0x36] = 0x9a
local_68[0x37] = 0x21
local_58[0x37] = 0x4f
local_68[0x38] = 0xaa
local_58[0x38] = 0xf5
local_68[0x39] = 0xcc
local_58[0x39] = 0xaf
local_68[0x3a] = 0x4a
local_58[0x3a] = 0x22
local_68[0x3b] = 2
local_58[0x3b] = 99
local_68[0x3c] = 0x2e
local_58[0x3c] = 0x42
local_68[0x3d] = 0xb0
local_58[0x3d] = 0xdc
local_68[0x3e] = 0xd4
local_58[0x3e] = 0xb1
local_68[0x3f] = 0x8c
local_58[0x3f] = 0xe2
local_68[0x40] = 0x49
local_58[0x40] = 0x2e
local_68[0x41] = 0x3d
local_58[0x41] = 0x58
local_68[0x42] = 0xa5
local_58[0x42] = 0x89
local_68[0x43] = 0x7e
local_58[0x43] = 0x1c
local_68[0x44] = 0x8e
local_58[0x44] = 0xfb
local_68[0x45] = 0x15
local_58[0x45] = 0x61
local_68[0x46] = 0xd9
local_58[0x46] = 0x86
local_68[0x47] = 0x41
local_58[0x47] = 0x28
local_68[0x48] = 0x98
local_58[0x48] = 0xec
local_68[0x49] = 0xef
local_58[0x49] = 0xb0
local_68[0x4a] = 0x90
local_58[0x4a] = 0xfd
local_68[0x4b] = 0x34
local_58[0x4b] = 0x5d
local_68[0x4c] = 0xa9
local_58[0x4c] = 0xce
local_68[0x4d] = 0xf
local_58[0x4d] = 0x67
local_68[0x4e] = 0x7c
local_58[0x4e] = 8
local_68[0x4f] = 0x4c
local_58[0x4f] = 0x13
local_68[0x50] = 0x45
local_58[0x50] = 0x27
local_68[0x51] = 0x7b
local_58[0x51] = 0x1e
local_68[0x52] = 0x6e
local_58[0x52] = 0x31
local_68[0x53] = 0x95
local_58[0x53] = 0xf4
local_68[0x54] = 0x3a
local_58[0x54] = 0x65
local_68[0x55] = 0x80
local_58[0x55] = 0xe7
local_68[0x56] = 0xcd
local_58[0x56] = 0xa2
local_68[0x57] = 0x7b
local_58[0x57] = 0x14
local_68[0x58] = 0xe2
local_58[0x58] = 0x86
local_68[0x59] = 0x9f
local_58[0x59] = 0xc0
local_68[0x5a] = 0x3c
local_58[0x5a] = 0x48
local_68[0x5b] = 0xcc
local_58[0x5b] = 0xa5
local_68[0x5c] = 0x6a
local_58[0x5c] = 7
local_68[0x5d] = 0xf8
local_58[0x5d] = 0x9d
local_68[0x5e] = 0x67
local_58[0x5e] = 0x38
local_68[0x5f] = 0x77
local_58[0x5f] = 3
local_68[0x60] = 0xf3
local_58[0x60] = 0x9c
local_68[0x61] = 0xa8
local_58[0x61] = 0xf7
local_68[0x62] = 0x55
local_58[0x62] = 0x39
local_68[99] = 0x65
local_58[99] = 0
local_68[100] = 0x15
local_58[100] = 0x74
local_68[0x65] = 0xa7
local_58[0x65] = 0xd5
local_68[0x66] = 0x87
local_58[0x66] = 0xe9
local_68[0x67] = 0xcb
local_58[0x67] = 0x94
local_68[0x68] = 0x89
local_58[0x68] = 0xe8
local_68[0x69] = 0x1d
local_58[0x69] = 0x7f
local_68[0x6a] = 0xcf
local_58[0x6a] = 0xa0
local_68[0x6b] = 0xd1
local_58[0x6b] = 0xa4
local_68[0x6c] = 0x1b
local_58[0x6c] = 0x6f
local_68[0x6d] = 0x79
local_58[0x6d] = 0x26
local_68[0x6e] = 0xe7
local_58[0x6e] = 0x86
local_68[0x6f] = 0x7d
local_58[0x6f] = 8
local_68[0x70] = 0x5c
local_58[0x70] = 0x28
local_68[0x71] = 0xf0
local_58[0x71] = 0x9f
local_68[0x72] = 0xfb
local_58[0x72] = 0x96
local_68[0x73] = 9
local_58[0x73] = 0x68
local_68[0x74] = 0x4a
local_58[0x74] = 0x3e
local_68[0x75] = 0x45
local_58[0x75] = 0x2c
local_68[0x76] = 7
local_58[0x76] = 0x69
local_68[0x77] = 0x74
local_58[0x77] = 0x13
local_68[0x78] = 0xcd
local_58[0x78] = 0x92
local_68[0x79] = 0x7f
local_58[0x79] = 0xb
local_68[0x7a] = 0x98
local_58[0x7a] = 0xf0
local_68[0x7b] = 0xb
local_58[0x7b] = 0x62
local_68[0x7c] = 0xaa
local_58[0x7c] = 0xd9
local_68[0x7d] = 4
local_58[0x7d] = 0x5b
local_68[0x7e] = 0xe
local_58[0x7e] = 0x7e
local_68[0x7f] = 0x70
local_58[0x7f] = 2
local_68[0x80] = 0x22
local_58[0x80] = 0x4d
local_68[0x81] = 0x11
local_58[0x81] = 0x72
local_68[0x82] = 0x9c
local_58[0x82] = 0xf9
local_68[0x83] = 0x72
local_58[0x83] = 1
local_68[0x84] = 0x3e
local_58[0x84] = 0x4d
local_68[0x85] = 0x7c
local_58[0x85] = 0x43
local_68[0x86] = 0x86
local_58[0x86] = 0xdf
local_68[0x87] = 0x1c
local_58[0x87] = 0x73
local_68[0x88] = 0x21
local_58[0x88] = 0x54
local_68[0x89] = 0x6e
local_58[0x89] = 0x31
local_68[0x8a] = 0x18
local_58[0x8a] = 0x75
local_68[0x8b] = 0x4e
local_58[0x8b] = 0x27
local_68[0x8c] = 0x30
local_58[0x8c] = 0x57
local_68[0x8d] = 0x3b
local_58[0x8d] = 0x53
local_68[0x8e] = 0xf
local_58[0x8e] = 0x7b
local_68[0x8f] = 0x2a
local_58[0x8f] = 0x75
local_68[0x90] = 0x26
local_58[0x90] = 0x48
local_68[0x91] = 0x6f
local_58[0x91] = 10
local_68[0x92] = 0xb8
local_58[0x92] = 0xdd
local_68[0x93] = 0x8e
local_58[0x93] = 0xea
local_68[0x94] = 0x4c
local_58[0x94] = 0x13
local_68[0x95] = 0x7f
local_58[0x95] = 0xb
local_68[0x96] = 0x22
local_58[0x96] = 0x4d
local_68[0x97] = 0xef
local_58[0x97] = 0xb0
local_68[0x98] = 99
local_58[0x98] = 7
local_68[0x99] = 0xa3
local_58[0x99] = 0xcc
local_68[0x9a] = 0xc2
local_58[0x9a] = 0x9d
local_68[0x9b] = 200
local_58[0x9b] = 0xa1
local_68[0x9c] = 99
local_58[0x9c] = 0x17
local_68[0x9d] = 0xd8
local_58[0x9d] = 0x87
local_68[0x9e] = 2
local_58[0x9e] = 0x60
local_68[0x9f] = 0x88
local_58[0x9f] = 0xed
local_68[0xa0] = 0x19
local_58[0xa0] = 0x7a
local_68[0xa1] = 0x5b
local_58[0xa1] = 0x3a
local_68[0xa2] = 0x78
local_58[0xa2] = 0xd
local_68[0xa3] = 0x7a
local_58[0xa3] = 9
local_68[0xa4] = 0x39
local_58[0xa4] = 0x5c
local_68[0xa5] = 0x8c
local_58[0xa5] = 0xd3
local_68[0xa6] = 0xbf
local_58[0xa6] = 0xd7
local_68[0xa7] = 0xc3
local_58[0xa7] = 0xa6
local_68[0xa8] = 0x19
local_58[0xa8] = 0x6b
local_68[0xa9] = 0x5d
local_58[0xa9] = 0x38
local_68[0xaa] = 0x87
local_58[0xaa] = 0xd8
local_68[0xab] = 0x9a
local_58[0xab] = 0xf3
local_68[0xac] = 0xd9
local_58[0xac] = 0xaa
local_68[0xad] = 0xa2
local_58[0xad] = 0xfd
local_68[0xae] = 0xea
local_58[0xae] = 0x8b
local_68[0xaf] = 0x7f
local_58[0xaf] = 0x20
local_68[0xb0] = 0xfd
local_58[0xb0] = 0x8d
local_68[0xb1] = 0xd0
local_58[0xb1] = 0xb1
local_68[0xb2] = 0xf9
local_58[0xb2] = 0x90
local_68[0xb3] = 0xb6
local_58[0xb3] = 0xd8
local_68[0xb4] = 0xf7
local_58[0xb4] = 0x91
local_68[0xb5] = 0x6e
local_58[0xb5] = 0x1b
local_68[0xb6] = 0xad
local_58[0xb6] = 0xc1
local_68[0xb7] = 0xaf
local_58[0xb7] = 0xf0
local_68[0xb8] = 0x5d
local_58[0xb8] = 0x35
local_68[0xb9] = 0xd8
local_58[0xb9] = 0xbd
local_68[0xba] = 0xca
local_58[0xba] = 0xb2
local_68[0xbb] = 0x29
local_58[0xbb] = 0x13
local_68[0xbc] = 0x8f
local_58[0xbc] = 0xbc
local_68[0xbd] = 0xb5
local_58[0xbd] = 0x87
local_68[0xbe] = 0xf
local_58[0xbe] = 0x6e
local_68[0xbf] = 0xbd
local_58[0xbf] = 0x8c
local_68[0xc0] = 0xce
local_58[0xc0] = 0xf8
local_68[0xc1] = 0xf8
local_58[0xc1] = 0x9a
local_68[0xc2] = 0x9d
local_58[0xc2] = 0xae
local_68[0xc3] = 0xcb
local_58[0xc3] = 0xaa
local_68[0xc4] = 0x18
local_58[0xc4] = 0x2f
local_68[0xc5] = 0xa0
local_58[0xc5] = 0xc5
local_68[0xc6] = 0x8f
local_58[0xc6] = 0xea
local_68[199] = 0xce
local_58[199] = 0xa8
local_68[200] = 0x30
local_58[200] = 8
local_68[0xc9] = 0x77
local_58[0xc9] = 0x13
local_68[0xca] = 0x5f
local_58[0xca] = 0x3a
local_68[0xcb] = 0x8a
local_58[0xcb] = 0xbb
local_68[0xcc] = 0xeb
local_58[0xcc] = 0xd9
local_68[0xcd] = 10
local_58[0xcd] = 0x3c
local_68[0xce] = 0x73
local_58[0xce] = 0x40
local_68[0xcf] = 0xda
local_58[0xcf] = 0xe2
local_68[0xd0] = 0x9e
local_58[0xd0] = 0xaf
local_68[0xd1] = 0x61
local_58[0xd1] = 0x53
local_68[0xd2] = 0x7f
local_58[0xd2] = 0x51
local_68[0xd3] = 0x51
local_58[0xd3] = 0x14
local_68[0xd4] = 0x20
local_58[0xd4] = 0x4e
local_68[0xd5] = 0x69
local_58[0xd5] = 3
local_68[0xd6] = 0x10
local_58[0xd6] = 0x7f
local_68[0xd7] = 0x69
local_58[0xd7] = 0x10
local_68[0xd8] = 0xf1
local_58[0xd8] = 0xd9
local_68[0xd9] = 0x26
local_58[0xd9] = 0x49
local_68[0xda] = 0x9e
local_58[0xda] = 0xec
local_68[0xdb] = 0x7d
local_58[0xdb] = 0x22
local_68[0xdc] = 0xf7
local_58[0xdc] = 0x99
local_68[0xdd] = 6
local_58[0xdd] = 0x69
local_68[0xde] = 10
local_58[0xde] = 0x7e
local_68[0xdf] = 0xb0
local_58[0xdf] = 0x99
local_68[0xe0] = 0x5f
local_58[0xe0] = 0x7e
local_68[0xe1] = 0x22
local_58[0xe1] = 0x5f


for i,j in zip(local_58,local_68):
    print(chr(i^j),end="")

#cvctf{i_apologize_for_such_a_long_string_in_this_checkin_challenge,but_it_might_be_a_good_time_to_learn_about_automating_this_process?You_might_need_to_do_it_because_here_is_a_painful_hex:32a16b3a7eef8de1263812.Enjoy(or_not)!}
```

# Micro Assembly

![micro asm](/assets/img/post_img/cvctf23_microasm.png)

Attached File : <a href="/assets/files/cvctf23/file.asm">file.asm</a>

The content of file.asm

```asm
main:
   PUSH %BP
   MOV  %SP, %BP
@main_body:
   SUB  %SP, $28, %SP
   MOV  $154, -28(%BP)
   MOV  $16, -24(%BP)
   MOV  $16, -20(%BP)
   MOV  $228, -16(%BP)
   MOV  $66, -12(%BP)
   MOV  $286, -8(%BP)
   MOV  $3, -4(%BP)
@if0:
   DIV  -28(%BP), $2, %0
   CMP  %12, $0
   JNE  @false0
@true0:
   DIV  -28(%BP), $2, %0
   MOV  %0, -28(%BP)
   JMP  @exit0
@false0:
@exit0:
   MUL  -24(%BP), $3, %0
   ADD  %0, $1, %0
   MOV  %0, -24(%BP)
   SHL  -20(%BP), $2, %0
   ADD  %0, $3, %0
   MOV  %0, -20(%BP)
@if1:
   DIV  -16(%BP), $3, %0
   CMP  %12, $1
   JNE  @false1
@true1:
   SUB  -16(%BP), $1, %0
   DIV  %0, $3, %0
   MOV  %0, -16(%BP)
   JMP  @exit1
@false1:
   DIV  -16(%BP), $2, %0
   MOV  %0, -16(%BP)
@exit1:
   SUB  -12(%BP), $2, %0
   MOV  %0, -12(%BP)
@if2:
   DIV  -8(%BP), $3, %0
   CMP  %12, $1
   JNE  @false2
@true2:
   SUB  -8(%BP), $1, %0
   DIV  %0, $3, %0
   MOV  %0, -8(%BP)
   JMP  @exit2
@false2:
   DIV  -8(%BP), $2, %0
   MOV  %0, -8(%BP)
@exit2:
   SHL  $11, -4(%BP), %0
   ADD  %0, $11, %0
   MOV  %0, -4(%BP)
   LEA  -28(%BP), %0
   MOV  %0, %13
   JMP  @main_exit
@main_exit:
   MOV  %BP, %SP
   POP  %BP
   RET 
```

Yeah, I know `Assembly`. There are `7` memory locations are initialized I thought that it was 7 character long flag. So, I started solving it by hand. 

Let me clearly explain the first `if` block.

```asm
@if0:
   DIV  -28(%BP), $2, %0     // 154/2 == 77
   CMP  %12, $0               // %12, 77 
   JNE  @false0
@true0:
   DIV  -28(%BP), $2, %0     // 154/2 = 77
   MOV  %0, -28(%BP)          //-28(bp) = 77 (154 --> 77)
   JMP  @exit0
@false0:
```

`-28(%BP)` is the location of value `154` and `$2` is just an integer.<br>`DIV 154,2` is stored in `%0`<br>
The result `77` is compared with the value present in `%12` location<br> I assumed It was true, So the `@true0` will be executed.<br> The true block is just doing DIV again and stored the value at `-28(%BP)`, which means the `158` is erased and `77` is stored inplace of it. 

By this way, I traced all the conditions. Most of the time I assumed the condition was true. Only when I got the result in decimal points marked it as false and excuted the false block.

Here is my rough work of the challenge.

```asm
main:
   PUSH %BP
   MOV  %SP, %BP
@main_body:
   SUB  %SP, $28, %SP
   MOV  $154, -28(%BP)
   MOV  $16, -24(%BP)
   MOV  $16, -20(%BP)
   MOV  $228, -16(%BP)
   MOV  $66, -12(%BP)
   MOV  $286, -8(%BP)
   MOV  $3, -4(%BP)
@if0:
   DIV  -28(%BP), $2, %0     // 154/2 == 77
   CMP  %12, $0               // %12, 77 
   JNE  @false0
@true0:
   DIV  -28(%BP), $2, %0     // 154/2 = 77
   MOV  %0, -28(%BP)          //-28(bp) = 77 (154 --> 77)
   JMP  @exit0
@false0:
@exit0:
   MUL  -24(%BP), $3, %0      // 16 * 3 = 48
   ADD  %0, $1, %0            // 48+1 = 49
   MOV  %0, -24(%BP)          // -24(bp) = 2 (16 -- > 49)
   SHL  -20(%BP), $2, %0      //-20(bp) = 16..  16<< 2 = 64
   ADD  %0, $3, %0            // 64 + 3 = 67
   MOV  %0, -20(%BP)          // -20(bp) = 67  (16-->67)
@if1:
   DIV  -16(%BP), $3, %0      // -16(bp) = 228  .. 228/3 = 76
   CMP  %12, $1               // %12, 1 
   JNE  @false1
@true1:
   SUB  -16(%BP), $1, %0      // 228 - 1 = 227
   DIV  %0, $3, %0            // 227/3 = 75.666
   MOV  %0, -16(%BP)          // 
   JMP  @exit1
@false1:
   DIV  -16(%BP), $2, %0      // 228/2 = 114
   MOV  %0, -16(%BP)          // -16(bp) = 114
@exit1:
   SUB  -12(%BP), $2, %0      // 66 - 2 = 64
   MOV  %0, -12(%BP)          // -12(bp) = 64
@if2:
   DIV  -8(%BP), $3, %0       // 286/3 = 95.333
   CMP  %12, $1         
   JNE  @false2
@true2:
   SUB  -8(%BP), $1, %0       // 286-1 = 285
   DIV  %0, $3, %0            // 285/3 = 95
   MOV  %0, -8(%BP)           // -8(bp) = 95
   JMP  @exit2
@false2:
   DIV  -8(%BP), $2, %0     // 286/2 = 143
   MOV  %0, -8(%BP)          // -8(bp) = 143
@exit2:
   SHL  $11, -4(%BP), %0      // 11 << 3 = 88
   ADD  %0, $11, %0           // 88 + 11 = 99
   MOV  %0, -4(%BP)           // -4(bp) =99
   LEA  -28(%BP), %0          // 77, %0
   MOV  %0, %13               // %13 = 77
   JMP  @main_exit
@main_exit:
   MOV  %BP, %SP
   POP  %BP
   RET 

cvctf{M1Cr@_c}
```

Later the values present in the memory locations from `-28(%BP)` to `-4(%BP)` converted these integers to ascii and The result was wrapped around cvctf{}.

> `Flag : cvctf{M1Cr@_c}`

***

# Misc

# Baby Calculator

![baby calc](/assets/img/post_img/cvctf23_babycalc.png)

`nc 20.169.252.240 4200`

A good challenge to do writeup. 

```sh
mj0ln1r@AHLinux:~/babycalc$ nc 20.169.252.240 4200
Welcome to the Baby Calculator! Answer 40 questions in a minute to get the flag.
The difference between your input and the correct answer must be less than 1e-6.
[+] Question 1:
Evaluate the integral of: 4/x from 3 to 7.
> Answer: 

```

It was asking us to do integral of `4/x from 3 to 7`, It was easy to do with `scipy` module of python

```python
from scipy.integrate import quad
def compute_integral(func, lower_limit, upper_limit):
    integral, error = quad(func, lower_limit, upper_limit)
    return integral
def f(x):
	return 4/x 
compute_integral(f,3,7)

```

But it was difficult to answer 40 questions in a minute. So, I used `pwntools` to automate this process.

This is my solution script.

```python
from math import cos, exp, pi,sin
from scipy.integrate import quad
from pwn import *

def compute_integral(func, lower_limit, upper_limit):
    integral, error = quad(func, lower_limit, upper_limit)
    return integral
def f(x):
    return eval(exp)
conn = remote('20.169.252.240',4200)
for i in range(40):
    conn.recvuntil(b'Evaluate the integral of: ')
    exp = conn.recvuntil(b' ').decode()
    r = conn.recvuntil(b'.').decode().strip('.')
    r = r.split(" ")
    ranges = [int(r[1]),int(r[3])]
    integral = compute_integral(f, ranges[0], ranges[1])
    print(integral)
    conn.recv()
    conn.sendline(str(integral).encode())
    if i == 39:
        print(conn.recv())

# cvctf{B4by_m@7h_G14n7_5t3P}
```

> `Flag : cvctf{B4by_m@7h_G14n7_5t3P}`

# Misc 

# OJail

![ojail](/assets/img/post_img/cvctf23_ojail.png)

Challenge

```sh
mj0ln1r@Linux:~/ojail$ nc 20.169.252.240 4201
        OCaml version 4.08.1

# 
```

Okay its time to learn `OCaml`. I quickly checked the Ocaml hello world program and tried it.

```sh
mj0ln1r@Linux:~/ojail$ nc 20.169.252.240 4201
        OCaml version 4.08.1

# print_string "hello world!\n";;
hello world!
- : unit = ()
# 
```
Yes, it worked. So, I found that with  `Sys.command` we can executed the system commands. I tried with `ls`. I found that there were a flag.txt but it was not the correct one. Then looked into `secret` directory and catted out the `flag-1d573e0faa99.json`

```sh
mj0ln1r@Linux:~/ojail$ nc 20.169.252.240 4201
        OCaml version 4.08.1

# Sys.command "ls";;
Dockerfile
build.sh
flag.txt
secret
- : int = 0
# Sys.command "ls secret";;       
flag-1d573e0faa99.json
- : int = 0
# Sys.command "cat secret/flag-1d573e0faa99.json";;
{
    "message": "Flag is here. OCaml syntax is easy, right?",
    "flag": "cvctf{J41L3d_OOOO-C4mL@@}"
}- : int = 0
# 

```

> `Flag : cvctf{J41L3d_OOOO-C4mL@@}`
 
***