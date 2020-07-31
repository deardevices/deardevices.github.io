---
layout: post
title: "How to Deal with a 4-bit Wrapping Counter"
---

Today's article will be about integers and twos' complement representation. It feels a bit like a follow-up on the [previous one]({% post_url 2020-07-03-twos-complement %}), where we did some experimenting with twos' complements on narrow integer types. This time we are going to think more about how twos' complement may *help us handling counter overflows*.

Imagine you've got a 4-bit counter in hardware, and your job is to periodically check the **difference between its current count and the one from the previous cycle**. This is useful, for example, to find out how much time has passed between cycles.

The issue with such a counter is that it will overflow at one point. You might get lucky and never have to deal with it -- if the value never reaches the maximum, for example.

Apart from this, calculating the difference will work as expected in most cases, while for others it won't. The table below shows some counter values, and the difference you would expect vs. what you will really get if you do the calculation.

| Count | PreviousCount  | Count - PreviousCount (Expected) | Count - PreviousCount (Actual) |
|-------| -------------- | -------------------------------- | ------------------------------ |
| 13    | -              | -                                | -                              |
| 14    | 13             | 1                                | 1                              |
| 15    | 14             | 1                                | 1                              |
| 0     | 15             | 1                                | **-15**                        |
| 1     | 0              | 1                                | 1                              |
| 2     | 1              | 1                                | 1                              |

In the last column, you can see the large negative difference at the point where the counter wraps from 15 to 0.

## How to fix it
To fix that unexpected value we got near the overflow, we might just handle it explicitly, like this:
```c
if (diff == -15)
  diff = 1
```

Well, this doesn't look very elegant, does it? Especially because we would probably want to also handle the opposite case, where the counter runs in negative direction, yielding a difference of `+15` near the overflow:

```c
if (diff == 15)
  diff = -1
```


## Better / more elegant solution?
This is a perfect occasion to look at how the twos' complement representation might help us. After all, this is how the operation is realized under the hood: instead of subtracting the second operand from the first, its twos' complement is added.

Maybe you've heard that you *simply don't need to care about overflows when using unsigned integers*. I have -- but honestly, I never thought much about what it was that *magically* took care of the overflow case for me.

To give you an example, let's dive right in, using some `uint8_t` numbers.

For 8-bit unsigned integers, the critical overflow happens at the value of 255: `255 - 254 = 1`, but `0 - 255 = -255`. Looking a bit closer, the actual subtraction is done like this:

```
  0    -  255
= 0x00 + ~0xff + 1
= 0x00 +  0x00 + 1
= 0x00 +  0x01
= 0x01
= 1
```

Look at this! Calculating the difference just seems to work! We get the expected result of `+1`.

## How to make it work for 4-bit integers

Now let's try the same experiment for numbers that don't fill up the full range of the data type: 4-bit numbers stored inside a `uint8_t` variable. As shown in the table above, the critical calculation, where the overflow gets in our way, would be `0 - 15`.

If we do this (in binary notation this time, because I hope it's easier to read), we'll end up with an interesting result:

```
  0          -  15
= 0b00000000 + ~0b00001111 + 1
= 0b00000000 +  0b11110000 + 1
= 0b00000000 +  0b11110001
= 0b11110001
```

If you look closely, you'll see that the magic *did happen*, just like in the example before: the expected result is contained in the last four bits!

So it seems the general solution is to mask out everything except the lower 4 bits: `0b11110001 & 0b00001111 = 0b00000001`. This is not a proof, of course -- but at least it is in line with [this answer on Stack Overflow](https://stackoverflow.com/a/3097602).

For the sake of completeness, let's check the logic for the non-wrapping case, e.g. `2 - 1 = 1`:

```
  2          -  1
= 0b00000010 + ~0b00000001 + 1
= 0b00000010 +  0b11111110 + 1
= 0b00000010 +  0b11111111
= 0b100000001
```

This is basically the same outcome as before: if we only care about the last 4 bits (by masking out all the others), we get our expected result of `1`.

## Summary
We have seen that we can safely calculate the difference of two (unsigned) counter values, and the overflow is automatically taken care of. This is valid as long as we can make sure the counter won't wrap multiple times.

If our number range is smaller than data type width, masking after calculating the difference will do the trick.
