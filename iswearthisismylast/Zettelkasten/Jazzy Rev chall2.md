2022-07-12 22:19
source: jazzy
tags: #research #reverse_engineering #cybersec #assembly #ctf


# Jazzy Rev chall1
Jazzy's ctf package rev chall1. used in UWCTF S22 (in the CTF its named chall2)

## Thought Process
Nothing much here

## Summary
When looking at source code, the flag existed as 
`#define FLAG "\x0cN\x11\x18\x13L \x1d\x06\x0bL \x07O\r N\x0c O\x1dJ\x1c*\rL"`

This is ASCII encoded, but not machine readable. 
The actual main function checks if input is equal to the FLAG after XOR-ing each character in input with 0x7f.

```
for(int i=0; i<strlen(argv[1]); i++ ){  
   if( (argv[1][i] ^ 0x7f) != FLAG[i] ){  
      printf("OOps!! Wrong password\n");  
      return 1;  
   }
```

Ran it in [[IDA]]. 
```
# The intersting parts
add     rax, rdx -> rdx is probably the counter? (i)
movzx   eax, byte ptr [rax] 
xor     eax, 7Fh -> xor
mov     ecx, eax
mov     eax, [rbp+var_14]
cdqe
lea     rdx, unk_40204D
movzx   eax, byte ptr [rax+rdx] -> grab the char in FLAG?
cmp     cl, al
```
In IDA, 0x7f existed as
```
movzx   eax, byte ptr [rax]
xor     eax, 7Fh
```

Which I assume first line assigns eax to the character currently comparing and next line performs xor. This value was then compared to the character at same position of our FLAG (saw in source code)

The FLAG was referred to as `unk_40204D`

Searching for this yielded:
```
                         unk_40204D      db  0Ch  ; DATA XREF: main+76↑o
.rodata:000000000040204E                 db  4Eh ; N
.rodata:000000000040204F                 db  11h
.rodata:0000000000402050                 db  18h
.rodata:0000000000402051                 db  13h
.rodata:0000000000402052                 db  4Ch ; L
.rodata:0000000000402053                 db  20h
.rodata:0000000000402054                 db  1Dh
.rodata:0000000000402055                 db    6
.rodata:0000000000402056                 db  0Bh
.rodata:0000000000402057                 db  4Ch ; L
.rodata:0000000000402058                 db  20h
.rodata:0000000000402059                 db    7
.rodata:000000000040205A                 db  4Fh ; O
.rodata:000000000040205B                 db  0Dh
.rodata:000000000040205C                 db  20h
.rodata:000000000040205D                 db  4Eh ; N
.rodata:000000000040205E                 db  0Ch
.rodata:000000000040205F                 db  20h
.rodata:0000000000402060                 db  4Fh ; O
.rodata:0000000000402061                 db  1Dh
.rodata:0000000000402062                 db  4Ah ; J
.rodata:0000000000402063                 db  1Ch
.rodata:0000000000402064                 db  2Ah ; *
.rodata:0000000000402065                 db  0Dh
.rodata:0000000000402066                 db  4Ch ; L
.rodata:0000000000402067                 db    0
```

This was definitely intersting. After digging around, It was found that h stood for bytes... Not too sure on that still but the 2 digits before h is actually ASCII in hexform after XOR-ing with 0x7f!! 
`\x0C` - Apply XOR again -> `\x73` - Hex to ASCII -> `s`
This can be known by focusing on the 2nd line. 
`4Eh == N` Means correlation. After investigation, `0x4E == ASCII N` !!! So I learned this was ASCII. 
After applying XOR to all the following and performing hex to ASCII,
we get `goose{s1ngl3_byt3_x0r_1s_0b5cUr3}`