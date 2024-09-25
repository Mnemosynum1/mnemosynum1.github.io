---
title: "Write-Up HTB Challenge Simple Encryptor"
date: 2024-09-24T16:06:52+02:00
draft: false
hidetoc: true
summary: " "
categories: 
    - WriteUp
tags:
    - HackTheBox
    - WriteUp
    - Reverse
---

Simple Encryptor is a reverse challenge created by Leeky on HackTheBox. The difficulty is very easy.

Challenge description:
`On our regular checkups of our secret flag storage server we found out that we were hit by ransomware! The original flag data is nowhere to be found, but luckily we not only have the encrypted file but also the encryption program itself.`

Once the zip archive has been unpacked, we're left with two files: a binary file named encrypt and the encrypted flag. Using the file command, we can look at the characteristics of the binary file:

``` bash
$ file encrypt 
encrypt: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0bddc0a794eca6f6e2e9dac0b6190b62f07c4c75, for GNU/Linux 3.2.0, not stripped
```

Okey it's an ELF-64 binary not stripped. Using Ghidra's decompiler, the code is pretty straightforward to understand. Below is an image of the annotated decompiled file.

![Decompiled code](/images/posts/WU_Simple_Encryptor/decompiled.png)

The program starts by opening the flag file, obtaining its size and allocating a buffer on the heap. The program will copy the content of the flag file into it. Then it initialises a seed to passed at the `srand` function.

The encryption process follows. In a loop that iterates over each byte of the buffer, the program first generates a random number and XORs it with the current byte at position `i`. It then generates a second random number, but this time only uses the lower three bits (`random_n2 & 7`), which results in a value between 0 and 7. Finally, it applies a bitwise left rotation to the XORed byte using this second random value.

Then it opens a new file and writes first the seed, then the encrypted flag.

Writing the function to decrypt is simple. We just have to repeat this operation but in the inverse order. We will first make right rotation with the second random number and then xor it with the first random number.

I wrote this C code:

```C
#include <stdio.h>
#include <stdlib.h>

#include <time.h>

#define ROTR(X,N) ((N) == 0 ? (X) : (((X) >> (N)) | ((X) << (8 * sizeof(X) - (N)))))

int main(int argc, char **argv) {

  if (argc != 3) {
    exit(1);
  }

  FILE *fd_encrypted = fopen(argv[1], "r");
  fseek(fd_encrypted, 0, SEEK_END);
  long len_encrypted = ftell(fd_encrypted);
  fseek(fd_encrypted, 0, SEEK_SET);
  len_encrypted -= 4;

  unsigned char *buffer = malloc((size_t)(len_encrypted + 1));
  int seed = 0;
  fread(&seed, sizeof(int), 1, fd_encrypted);

  fread(buffer, len_encrypted, 1, fd_encrypted);
  srand(seed);

  for (int i = 0; i < len_encrypted; i++) {
    int rand_n1 = rand();
    unsigned char rand_n2 = (unsigned char)rand() & 7;
    buffer[i] = ROTR(buffer[i], rand_n2);
    buffer[i] = buffer[i] ^ rand_n1;
  }
  buffer[len_encrypted] = '\0';

  printf("Flag = %s\n", buffer);

  FILE *fd_decrypted = fopen(argv[2], "w");
  fwrite(buffer, len_encrypted, 1, fd_decrypted);
  fclose(fd_encrypted);
  fclose(fd_decrypted);
  free(buffer);

  return 0;
}
```
