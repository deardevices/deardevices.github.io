---
layout: post
title: "KNX Link-Layer Ack: A brief analysis."
draft: true
---

KNX is a two-wire bus for building automation. It operates at a pretty low bitrate, which makes it a good candidate for analyzing communications using a cheap logic analyzer. This article is about the Acknowlegement procedure at the KNX Link Layer.

As with many communication stacks, KNX[^1] is split into several layers. The Link Layer (Layer 2) is the one that sits on top of the Physical Layer (Layer 1). Its job is to make sure that data is exchanged correctly between KNX devices. An important aspect of this is the Acknowlegdgement procedure: Every KNX telegram is acknowledged by one or more receivers when transmitted over the wire.

The image below shows the setup for experimenting with Link-Layer messaging. You can see how horizontal communication (logical, between Link Layers) is captured indirectly on the two-wire medium using a logic analyzer.

<p align="center">
   <img alt="Experiment Setup" src="/assets/knx/experiment-setup.jpg" width="80%" />
</p>

## Immediate ACK
Acknowledgment on the KNX Link Layer is also called *Immediate ACK (IACK)* in KNX jargon, presumably to differentiate it from other ack methods on the upper layers. Here's how it is supposed to work:

1. Sender transmits KNX telegram
2. [No communications for a certain period of time]
3. Receiver transmits acknowledgment character
4. Sender evaluates acknowledgment

Here's how this looks on the wire, captured from a logic analyzer:
![](/assets/knx/ack-on-logicanalyzer.png)

This recording shows a KNX telegram from address `0x11CF (1.1.207)` to group address[^2] `0x0901 (1/1/1)`. It is followed by an *ACK* character (`0xCC`), issued by the *receiving* device. The telegram consists of nine individual *characters*, each of which is framed by a Start, Stop, and Parity bit, similar to [RS-232](https://en.wikipedia.org/wiki/RS-232).

## Missing Acknowledgment!
What would happen if there was no positive acknowledgment after the sender transmitted the telegram? This situation is shown in the following picture. You can see four KNX telegrams, with nine characters each. The first one is identical to the one in the diagram above.
![](/assets/knx/no-ack-on-logicanalyzer.png)

Details of the frame are not visible, due to the zoom level chosen. What we *can* see, however, is the missing *ACK* character after each of the telegrams. Because there was no *ACK* after transmitting the first telegram, the sender started another attempt. After *three trials*, it gave up.

Actually, I don't know if that exact number of retries is mandatory -- at least that's what I've observed.

Now let's zoom in a bit to highlight the beginning of each telegram.

![](/assets/knx/no-ack-on-logicanalyzer-repeat-bit.png)

It becomes clear that there is a tiny difference between the original telegram and the repeated ones: the first character (also called *Control Field*) has a difference in bit 5:
```
Bit#     7654 3210
0xBC = 0b1011'1100
0x9C = 0b1001'1100
```

Bit #5 is the 'repeat' flag (who would have guessed that?). Its logic is inverted though: If it's set, that means *not repeated*.

If we think about it, the repeat flag provides useful information for layers further up the KNX stack: The telegram might have been processed already, so any repeated one might better be ignored in that case. Especially if we think about more than one device listening to a single group address, this seems to make sense.

## Several receivers, a single response
So far, we have looked at 1-to-1 relationships between one sender and one receiver. However, it is the very job of the KNX Link Layer to deliver telegrams from one sender to N receivers.

Let's think about a situation with three devices connected to the bus: one sender and two receivers. The sender transmits a telegram to group address `1/1/1` -- an address both receivers are configured to listen to.

Now, imagine that one receiver acknowledges the telegram (by transmitting an *ACK* character at the right time), while the other doesn't respond at all. Do you think that, from the sender's point of view, this is enough for the entire transmission to *succeed*?

It turns out that, due to the encoding of the bits (on layer 1, the Physical Layer), it is sufficient if only a single receiver acknowledges the telegram. This works as long as the other receiver simply stays passive on the bus during that time.

The implication of this is that a successful acknowledgment only tells the transmitting device that **at least one** device has received the telegram correctly.

## Conflicting Opinions

<p align="center">
   <img alt="Different Opinions Meme" src="/assets/knx/different-opinions.jpg" width="400" />
</p>

Alright, it's sufficient to get a response from a single receiver. What if one of the receivers actively reported a *negative* acknowledge instead of not reacting at all -- perhaps due to a checksum error it's seeing in the datagram?

In addition to the `ACK` character we've seen already, there are two other possible acknowledge characters a receiver may write to the bus: `NACK (0x0C)` and `BUSY (0xC0)`.

So let's look at the case where receiver A issues an `ACK`, while receiver B puts a `NACK` character onto the bus.

At this point, we need to consider how each bit is represented on the wire. This is the job of the Physical Layer. The short story is: For transmitting a `0` bit, the transmitter applies a certain load to the bus line for some microseconds. This makes the bus voltage drop and restore after a certain time. For transmitting a `1` bit, it simply does nothing during that period.

Interestingly, this scheme implies that `0` has priority over `1`: As soon as device A decides to generate such a voltage drop, there is no way for device B to issue a `1` (because just staying passive won't change anything, right?). In fact, the result on the bus is the logical *AND* of the bits written by all individual devices, as shown in the listing below.

```
  NACK 0x0C 0b00001100
& ACK  0xCC 0b11001100
= NACK 0x0C 0b00001100
```

This demonstrates that `NACK` overrides `ACK` on the bus.

## Conclusion
We have seen KNX Group Acknowledgment in action: A single ack response is enough for the sending device to be confident in the success of the transaction. Also, we have observed three different acknowledgment types overriding each other: Even with many receivers on the bus, a single device can cause a set of retries by publishing a *NACK* character.

## References
- [Serial Data Transmission and KNX Protocol](https://c3.chipkin.com/assets/uploads/imports/resources/Serial%20Data%20Transmission%20and%20KNX%20Protocol%2005_Serial%20Data%20Transmission_E0808f.pdf)
- [Building Automation](https://books.google.de/books?id=pxPq2CZSVooC&pg=PA82&lpg=PA82&dq=dominant+recessive+bus+knx&source=bl&ots=i7FnKmZh-N&sig=ACfU3U390R0J7kiyu7EnAC31jagjYvpMUA&hl=en&sa=X&ved=2ahUKEwi0k8nwzYbnAhWSzqQKHXUhDHcQ6AEwCnoECAoQAQ#v=onepage&q=dominant%20recessive%20bus%20knx&f=false)

[^1]: Strictly speaking, it is *KNX-TP* (twisted-pair) we're dealing with here -- there's also powerline and wireless variants.
[^2]: You can tell the telegram is going to a group address because 0xD1 has the most-significant bit set.
