---
title: "Write-Up FCSC2019 Not So Fat"
date: 2024-09-24T22:18:47+02:00
draft: false
hidetoc: true
summary: " "
categories: 
    - WriteUp
tags:
    - FCSC2019
    - WriteUp
    - Forensic
---

- Capture the flag: FSCS2019
- Categories: Forensic
- Title: Not so FAT
- Difficulty: intro
- Author: alx
- Description: `I have deleted the flag by mistake: could you recover it for me?`
- [Link](https://hackropole.fr/en/challenges/forensics/fcsc2019-forensics-not-so-fat/)

We start with a file named `not-so-fat.dd`. Using the command `file` we can determine the format:

```bash
$ file not-so-fat.dd 
not-so-fat.dd: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", sectors/cluster 4, reserved sectors 4, root entries 512, sectors 32768 (volumes <=32 MB), Media descriptor 0xf8, sectors/FAT 32, sectors/track 32, heads 64, serial number 0x3be84c04, unlabeled, FAT (16 bit)
```

This file is a disk image with a DOS/MBR boot sector and formatted as a FAT16 file system. We can mount this disk image with `mount` in read-only to avoid compromising file integrity.

```bash
$ mkdir mnt_not_so_fat
$ sudo mount -t vfat -o ro not_so_fat.dd mnt_not_so_fat
$ ls -lha mnt_not_so_fat
total 16K
drwxr-xr-x. 2 root  root  16K  1 janv.  1970 .
drwxr-xr-x. 1 lucas lucas 156 24 sept. 19:58 ..
```

The disk appears to be empty but if we check strings, there is file called `flag.txt`.

```bash
$ strings not-so-fat.dd 
mkfs.fat
;NO NAME    FAT16   
This is not a bootable disk.  Please insert a bootable floppy and
press any key to try again ... 
IEUYRJW    
LAG    ZIP 
flag.txtUT
flag.txtUT
```

This file appears to be deleted as mention in the description. However,  We can recover this file using some appropriate tools. This process is known as [file carving](https://en.wikipedia.org/wiki/File_carving). On Linux-based operating systems there are tools to perform this operation, such as Autopsy, Foremost or Photorec. Personally, I used this last one to carry out this challenge.

With this, we an encrypted zip file. So using `john` and `zip2john` we can try to crack it:

```bash
$ zip2john recup_dir.1/f0000104_flag.zip > hash.john
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.john 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Note: Passwords longer than 21 [worst case UTF-8] to 63 [ASCII] rejected
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
password         (f0000104_flag.zip/flag.txt)     
1g 0:00:00:00 DONE (2024-09-24 19:45) 33.33g/s 273066p/s 273066c/s 273066C/s 123456..total90
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Okay I should have known :'). Using this password we get the flag:

```bash
$ unzip -P password recup_dir.1/f0000104_flag.zip 
Archive:  recup_dir.1/f0000104_flag.zip
 extracting: flag.txt 
$ cat flag.txt
ECSC{eefea8cda693390c7ce0f6da6e388089dd615379}
```

Easy-Peasy.