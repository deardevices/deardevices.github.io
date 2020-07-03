---
layout: post
title: "Tinkering with signed integers in C"
---

Recently I came across 4-bit signed integers. While I just wanted to make sense of them somehow in the first place, I thought that's a nice opportunity to dig deeper.

# Signed, but positive
So, let's say we've got a four-bit signed number, `0111` in binary and, let's assume this value comes from some peripheral register. In addition, we have a datasheet that does not clearly describe the binary representation of this number.

They do define, however, the position of the sign bit: it's the most significant one (the *zero* on the left, in this case). It's fair to assume that the number is represented as either [ones'](https://en.wikipedia.org/wiki/Ones%27_complement) or [twos' complement](https://en.wikipedia.org/wiki/Two%27s_complement). If that's true, a value of zero for the sign bit means *positive*. Therefore, we can interpret this number simply as it is: `7`. We may assign it to an `int8_t` right away.

Our job is certainly going to be trickier if the number is negative. Let's now assume a binary value of `1111`.

# What representation is it?
First of all, the exact format is now really something we want to know: the value would translate to `-0` in ones' complement but would mean `-1` in twos' complement representation (see the last row of the table below).

Binary | Ones' Complement | Twos' Complement | Sign bit & value ("naïve")
  ---    | ---                    | ---                       |
 `0000`  |  0                     |  0                        |  0
 ...     | ...                    | ...                       | ...
 `0111`  |  7                     |  7                        |  7
 `1000`  | -7                     | -8                        | -0
 `1001`  | -6                     | -7                        | -1
 `1010`  | -5                     | -6                        | -2
 `1011`  | -4                     | -5                        | -3
 `1100`  | -3                     | -4                        | -4
 `1101`  | -2                     | -3                        | -5
 `1110`  | -1                     | -2                        | -6
 `1111`  | -0                     | -1                        | -7

After consulting the datasheet again, let's assume we come across the allowed range for that register. It reads: *(-8) through (+7)*.

This is a great clue if you compare it to the table above (focus on the first two columns): twos' complement is the only representation out of the three that doesn't have two different values for *zero*, and therefore allows a negative limit of (-8), instead of (-7). That's a strong indication that we are dealing with twos' complement.

You might be wondering what the third column is for. I've added it to show a somewhat "naïve" representation, where you turn positive numbers negative by simply putting a '1' bit in front of them. I think I've seen this kind of representation used occasionally; but because of its lack of the *complement* feature it's generally harder for computers to deal with them. Well, this was just a side note -- back to twos' complement.

Now that we know all that, we can deduce the actual value by applying the conversion rule for twos' complement:
```
1111 | 'not'
0000 | +1
0001
```

By flipping each bit and adding one, we get the result: It's a *-1*!

This is pretty good so far. However, to be able to do further calculations we would either need either native support for 4-bit signed integers or a way to convert the number to some type our platform supports.

# How to convert to `int8_t`?
Assuming we obtain the original value as a `uint8_t`, we may try to cast it to a signed type:
```
uint8_t input_value = 0x0f;
int8_t output_value = (int8_t)input_value;
```

Printing this value shows this cannot be right: the number is interpreted as **+15**. This is certainly not what we expected.

It seems we need to think more about the twos' complement representation. From [Twos' Complement (Wikipedia)](https://en.wikipedia.org/wiki/Two%27s_complement):
> The two's complement of an N-bit number is defined as its complement with respect to 2<sup>N</sup>. For instance, for the three-bit number 010, the two's complement is 110, because 010 + 110 = 1000

This applies to the 4-bit numbers in our example: Adding (+1) and (-1) yields 2<sup>4</sup> (in binary: `0001 + 1111 = 10000`).

So, if we want our negative number to be represented correctly as an 8-bit signed type, we need to adjust it in a way that it *follows the rules*: We know that it's (-1), so adding it to (+1) should result in 2<sup>8</sup> (`1'0000'0000`). Currently, this doesn't hold true: `0000'0001 + 0000'1111 = 0001'0000`.

Apparently there needs to be an overflow to bit #8 when adding (-1) and its complement. How could `0000'1111` be modified to make this happen? Well, if bits #7 through #4 were set before adding, this would work: `0000'0001 + 1111'1111 = 1'0000'0000`.

That seems to be the (general) solution: Replicate the sign bit of the input value to all higher bit positions of the target type, up to the most-significant bit.

# Conclusion
You can handle negative integers of 'narrow' (sub-8bit) types smoothly as `int8_t` if you adjust them so that they keep acting as a complement to their positive counterpart.

Let me know what you folks think (twitter link below)!
