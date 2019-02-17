---
layout: post
image: /assets/bitshift.png
---

Recently someone asked me about C's bitwise shift operator, specifically about what it does to **signed** integers. To be honest, until then I haven't even considered applying the left or right shift operator to anything else than unsigned numbers. So let's figure out what this is about.

 
## Unsigned Integers
First, we'll consider the unsigned case. Why would you want to apply a bit shift to an unsigned number in the first place? There are two motivations I can think of right now:
 
1. Bit manipulation, e.g. setting bit #5 of a register `MY_REG = MY_REG | (1<<5)`. Here you treat the variable MY_REG as a sequence of bits, rather than an actual number.

2. Multiplication / division by two: In many cases, this looks like some kind of *optimization* to me.

In either case, all bits are rotated by one position and a zero bit is put into the emerging gap. This is called a [Logical Shift](https://en.wikipedia.org/wiki/Logical_shift).

What can go wrong here? As long as you don't shift out any 1 value bit at the left, everything seems to be fine. Otherwise, the result gets reduced by modulo 2^N (due to the overflow), which is probably not what the programmer would expect. For example, the following 4-bit number is bit-shifted to the left two times:

```
0100b = 4
1000b = 8 -> as expected
0000b = 0 -> overflow: 8 * 2 = 16; reduced modulo 2^4 = 0
```

## Signed Integers
Now, what about **signed** numbers? I find it hard to think of them as a sequence of characters, as the Most-Significant Bit (MSB) has a special meaning here: it indicates the sign of the number represented by the remaining bits.

According to the reasoning from above, only multiplication/division is left as a motivation for using the shift operator in this case.

Let's first apply a **left shift** to a negative number (in two's complement) as an example:
```
1111b = (-1)
1110b = (-2)
```

This is not very surprising: a zero bit has been shifted in at the right and the number is still negative, so the result is a multiplication by 2.

For other numbers, the situation may be different. Let's do a left shift again:
```
1000b = (-8)
0000b = 0
```

This is unexpected -- just as before with unsigned numbers. Even worse, if we take a look at the C standard[^1], the result of this operation is undefined.

A change in behavior compared to unsigned operands can be observed by applying a **right shift**:
```
1110 = (-2)
1111 = (-1)
```

What happened here? Other than in the unsigned example, the MSB is still one; no zero bit has been shifted in at the left. It turns out this approach is called an [Arithmetic Shift](https://en.wikipedia.org/wiki/Arithmetic_shift) and the compiler[^2] applies it automatically, **based on the signedness of the operand**.

We can verify that this also works with positive signed numbers, where the zero MSB is preserved:

```
0010 = 2
0001 = 1
```

(All examples verified with GCC 8.)

## Conclusion & References
While the behavior of the right shift operator on signed integers looks promising, unfortunately the C standard states that the result is implementation-defined behavior.

So my conclusion is to not apply the bit shift operators on signed operands *at all*. There are distinct operators for multiplication and division after all, why not just use them instead?

I would love to know your opinion on this topic. Feel free to add a comment below!


[^1]: Chapter 6.5.7 in the [C99 Standards Document (publicly available)](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf)
[^2]: [GNU C Manual - Bit Shifting](https://www.gnu.org/software/gnu-c-manual/gnu-c-manual.html#Bit-Shifting)
