# Sequence

```
int in, out;

int g(int a, int b)
{
 return (a == b ? 1337 : -1);
}

void f()
{
  
/*INPUT*/

}

int main()
{
 const int N = 6;
 int values[N] = {1100, 1101, 1104, 1109, 1116, 1125};
 for (int i = 0; i < N; i++)
 {
   in = i; f();
   if (g(out, values[i]) != 1337) return 0;
 }
 victory();
 return 0;
}
```

```
Your main objective is to call the victory function.
You must put your code:
- Max 14 chars.
- You can't use: "main", "victory", "asm", "#", "_", "(".
```

## solution

### bad solution

- trivial: `out=in*in+1100;` (but 15 chars)
- good idea, but too long: another integer notation, like: 1e3 or 'L'

### working solution

- get stack address and overwrite N. solve for f
- easiest way to get the stack address is: `int a;unsigned stack_addr=(unsigned)&a;`;
- also you need to find a location in memory which is 0. global variables are initialized to 0 in C.

So the format is:
```
int a;*(&a+STACK_ADDR_RELATIVE_OFFSET)=POINT_TO_A_ZERO_VALUE;
// for example
int a;*(&a+3)=9;
```
which is violating the no `(` rule.
So do some tricks with `[]`. Using `[]` you can swap the variable and the offset, like, `a[5]` and `5[a]`. So you can change the predecence with it, so.
```
int a;3[&a]=9; // exactly 14 chars
```

- 3 and 9 are just guessed values
- by writing a for loop going over 0..9 for both numbers will find the final solution. The exact offsets and values can highly depend on the exact compiler version/options and architecture.

final solution:
```
int a;7[&a]=7;
```

