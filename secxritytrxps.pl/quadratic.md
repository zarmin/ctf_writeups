# Quadratic

```
#include<cstdio>
#include<cstdlib>
#include<cmath>

int main()
{
	long long a,b,c,d;
	printf("Quadratic equation solver!\n");
	printf("ax^2+bx+c=0, a!=0\n");
	printf("Enter a: ");
	scanf("%lld",&a);
	printf("Enter b: ");
	scanf("%lld",&b);
	printf("Enter c: ");
	scanf("%lld",&c);
	if(a==0)
	{
		printf("Idiot!!! a != 0 !!!\n"); // You fail :(
		return 0;
	}
	d=(b*b)-(4*a*c);
	if(d<0) 
	{
		printf("No solutions.\n"); // You fail :(
		return 0;
	}
	d=sqrt(d);
	if(d)
	{
		printf("Solutions: %lld and %lld.\n", // You fail :(
			(-b-d)/a/2,
			(-b+d)/a/2
		);
	}
	else
	{
		printf("Solution: %lld.\n", // You fail :(
			(-b)/a/2
		);

	}
	return 0; // You fail as well
}

Your main objective is to stop(crash) the execution before program writes the solutions.
```

## Possible parts of a crash 

- `sscanf` read crash -> *nothing wrong with it*
- `sqrt` (long long to double conversion, double to long long conversion) -> *sqrt won't crash on negativ argument*
- `printf` -> *nothing wrong with it*
- `SIGFPE` during some floating point error (overflow, underflow, division by zero, invalid value, etc.) -> *promising!*

## Why SIGFPE can happen?

Division by zero is trivial, but it won't happen, because `a` can't be 0 when reaches the division part.

This is from gnu.org and it's very promising.

```
In C, signed integer overflow leads to undefined behavior. However, many programs and Autoconf tests assume that signed integer overflow after addition, subtraction, or multiplication silently wraps around modulo a power of two, using two's complement arithmetic, so long as you cast the resulting value to an integer type or store it into an integer variable. Such programs are portable to the vast majority of modern platforms. However, signed integer division is not always harmless: for example, on CPUs of the i386 family, dividing INT_MIN by -1 yields a SIGFPE signal which by default terminates the program. Worse, taking the remainder of these two values typically yields the same signal on these CPUs, even though the C standard requires INT_MIN % -1 to yield zero because the expression does not overflow.
```

`INT_MIN` also working with int64.

But it only work on i386, the CPU must be 32bit. So bring up qemu for testing and install a debian 5 (from archive) on an emulated CPU (Pentium III).

```
qemu-img create hd_img.img 10g
qemu-system-i386 -cdrom ~/Downloads/debian-10.4.0-i386-netinst.iso -hda hd_img.img -boot d -m 2g -cpu pentium3-v1
```

VMware / VirtualBox / KVM or any other virtualization is not good, because it will still run on host's CPU which is 64 bit.

So, either `(-b-d)/a/2` or `(-b+d)/a/2` should be in a `INT64_MIN / -1` form.

- `a` can be -1 without any issues
- `d` muts be 1, because the maximum positive value of `b` is 9223372036854775807, which become -9223372036854775807, which is off-by-one to `INT64_MIN`, also `d` will be the result of `sqrt`.

```
1 = sqrt((b*b) - (4*(-1)*c))
```

9223372036854775807 * 9223372036854775807 will be 1, so to get 1, c must be 0.

## Solution

```
a = -1
b = 9223372036854775807
c = 0
```



