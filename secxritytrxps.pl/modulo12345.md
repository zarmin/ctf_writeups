# Mod12345

```
Your task is to write a function, that returns given parameter modulo 12345.
You must put your code:
- Max 200 chars.
- You can't use: "main", "asm", "modulo12345", "%", "/", "#", "&", "_", "(", "for", "while", "do", "goto".
```

More info: https://arxiv.org/pdf/1902.01961.pdf

Alternative modulo without division, loop, modulo operators.

```
unsigned int mod12345(unsigned int number)
{
  unsigned long long i=number;
  unsigned j=i*0x53c1df1dULL>>32ULL;
  j+=number-j>>1;
  j>>=0xd;
  return number-j*0x3039;
}
```

for an easier solution write the function in C and compile it with GCC and -O3, the disassembly will contain this "optimized" fix modulo algorithm.

```
# mod.cpp
int mod12345(unsigned number) {
  return number % 12345;
}
```

```
g++ a.cpp -O3 -c
objdump -disassemble -x86-asm-syntax=intel a.o

a.o:	file format Mach-O 64-bit x86-64


Disassembly of section __TEXT,__text:

0000000000000000 __Z11modulo12345j:
       0: 55                           	push	rbp
       1: 48 89 e5                     	mov	rbp, rsp
       4: 89 f8                        	mov	eax, edi
       6: 89 f9                        	mov	ecx, edi
       8: 48 69 c9 1d df c1 53         	imul	rcx, rcx, 1405214493
       f: 48 c1 e9 20                  	shr	rcx, 32
      13: 89 fa                        	mov	edx, edi
      15: 29 ca                        	sub	edx, ecx
      17: d1 ea                        	shr	edx
      19: 01 ca                        	add	edx, ecx
      1b: c1 ea 0d                     	shr	edx, 13
      1e: 69 ca 39 30 00 00            	imul	ecx, edx, 12345
      24: 29 c8                        	sub	eax, ecx
      26: 5d                           	pop	rbp
      27: c3                           	ret
```

And reimplement it in C code or just use [snowman](https://derevenets.com/).


