# C++ Trap #8

```
#include <cstdio>
int main()
{

/*
INPUT
*/

  return 0;
}
```

```
Your main objective is to write the text "Hello World!" to standard output.
You must put your code:
- Max 42 chars.
- You can't use: "main", "asm", "puts", "printf", "putc", "write", "builtin", "#", "*", "&", "_", ";", "%", "\".
```

## Approaches

### Puts with define

You can write # as trigraphs by ??=, and concatenate defined parts with ##.

So
`??=define a pu??=??=ts`

### Call function without semicolor

`if(a("Hello World!)) return 0`

### 44 char solution

```
??=define a pu??=??=ts
if(a("Hello World!))
```
22+21+1 (newline) = 44 char solution, not good!

### Usable functions to print

- std::cout: <iostream> need to be included (not possible)
- putw: possibly good!

### putw

Possible solution:

```
for(int i:"Hello World!")putw(i,stdout)
```

- Cant use semicolon and the next statement (`return 0`) will break the whole solution

## Solution - the right approach

- `perror`
- you need to swap the `stderr` with `stdout`
- `perror` is returning void, so you need to use the comma operator to make an expression for if

```
if(stderr=stdout,perror("Hello World!"),1)
```

So the whole `main` function will become

```
int main()
{
  if(stderr=stdout,perror("Hello World!"),1)
  return 0;
}
```

