---
date: 2024-04-22
categories:
  - CRC
  - PPP
  - COBS
---

# Error Cascade in PPP COBS

Consistent Overhead Byte Stuffing (COBS) is used to frame byte-oriented communications and is described in detail in [Wikipedia](https://en.wikipedia.org/wiki/Consistent_Overhead_Byte_Stuffing) and elsewhere.
COBS is used as a lower overhead alternative to HDLC Byte Stuffing.

## COBS Overview

A short explanation of COBS is that it replaces each `0x00` encountered in the data
with a linked list of the distances between each `0x00`.
Since the links can never be `0x00`,
the only overhead is the insertion of COBS Link-Bytes
to bridge the gaps between those `0x00` values more than 255 bytes apart.

COBS treats data exhausted of (or not containing) `0x00` as having a final `0x00` at the end of data.
This dictates the repeated insertion of COBS Link-Bytes until the final link,
which sets the location of the `0x00` used as the Frame Terminator thus _sealing_ the COBS frame.

Each COBS Link-Byte adds overhead at a consistent _worst-case rate_ of 1/255 of the message length.
This is much better than the _worst-case rate_ for HDLC Byte Stuffing, which is twice the message length.

!!! Overhead

    HDLC Byte Stuffing overhead is very sensitive to message content.
    Uniformly distributed random data (eg. encrypted data), achieves an average overhead of 2/256.
    However, high overhead is associated with data contain many HDLC _Frame Sequence_ (`0x7e`) or _Control Escape_ (`0x7d`) bytes.

    COBS, by contrast, is **not** very sensitive to message content.
    The worst and average overhead for random data is the same.
    The lowest overhead is the reciprocal of the message length, which is achieved by messages contain 0x00 at least once every 255 bytes.

## COBS-Checksum Interactions

COBS is simply a framing scheme and does not define checksum requirements per se.

Nevertheless a COBS receiver can correctly frame COBS encoded data 99.6% of the time due to the self-sealing nature of the frame.
Undetected errors can still occur, but the frames themselves should be generally identifiable.

Things become more complex when a checksum is added to detect data corruptions.

If the checksum contains `0x00` then it will either need to be placed outside the frame or encoded so that it can be placed within the frame.
Placing the checksum outside the frame introduces problems with frame synchronisation and should be avoided except under controlled conditions.
COBS encoding the checksum adds immediate overhead in the form of a Link-Byte and generally complicates the process.

## PPP COBS Checksums

Stuart Cheshire presented COBS for his [PhD](http://stuartcheshire.org/papers/Dissertation.ps),
and originally at [SIGCOMM '97](http://stuartcheshire.org/papers/COBSforSIGCOMM.ps).
The focus for these and other discussions of COBS was around the reduced overhead compared to HDLC-style Byte Stuffing.

The initial COBS application was intended for PPP,
which can be seen in the [PPP COBS Proposal](https://www.ietf.org/archive/id/draft-ietf-pppext-cobs-00.txt)
written around the same time as his PhD.
PPP itself is described in [RFC 1662](https://datatracker.ietf.org/doc/html/rfc1662.html)
and **does** specify the use of a "... Frame Check Sequence for error detection":

> The FCS field is calculated over all bits of the Address, Control,
> Protocol, Information and Padding fields, not including any start
> and stop bits (asynchronous) nor any bits (synchronous) or octets
> (asynchronous or synchronous) inserted for transparency.  This
> also does not include the Flag Sequences nor the FCS field itself.
>
> When octets are received which are flagged in the Async-
> Control-Character-Map, they are discarded before calculating
> the FCS.

The _Async-Control-Character-Map_ is outside the scope of present discussions,
aside from noting that it includes the HDLC _Frame Sequence_ (`0x7e`) and
_Control Escape_ (`0x7d`) used to perform Byte Stuffing.

However, in the PPP COBS Proposal the self-sealing nature of COBS Frames leads to the observation that:

> If one of the code byte octets is lost or corrupted, then the block
> will be miscounted.  This is likely to ultimately result in the
> inclusion of the trailing 7E within the data portion of an erroneous
> COBS block.  The receiver will detect this as either an error (if it
> does not support preemption) or as a preempted message.  In the
> latter case, the falsely preempted message will be discarded when the
> next true preemption occurs.  If the 7E still falls on a natural
> boundary between COBS blocks, then COBS will not detect the error,
> but the standard Frame Check Sequence (FCS) will be used to detect
> the corruption.

Later in an example implementation we find the following comment:

``` c
/*
* It turns out to be horribly complicated to support both the
* scan-ahead functions and the CRC calculation at the same time
* since the last byte of the CRC (or even both bytes) may be 00.
* Thus, it is necessary in this implementation to do the CRC
* first and the COBS encoding second in two separate steps.  If
* the COBS output were fed into a simple linear output buffer
* big enough to hold the largest packet, then this could be
* greatly simplified.
*/

```

So PPP COBS clearly favours calculating the FCS as a CRC applied to the PPP packet
**prior** to COBS encoding since the CRC may itself contain `0x00` values
that would then need to be Byte Stuffed to ensure frame delineation.

PPP COBS is not alone in this regard.

**Every** COBS recommendation and implementation I have seen chooses to append a checksum to the data prior to COBS encoding.
While this is much simpler, it can significantly reduce the strength of the checksum as we shall see.

## COBS Error Cascade

When any link in a COBS chain is corrupted, the COBS decoder will incorrectly decode the next and subsequent links.
A single COBS link error can therefore cascade into multiple errors at a rate of 8 bits per link (on average for uniform random data).

When the underlying data is long or contains frequent `0x00` values, many corruptions will occur due to the cascade.

If a checksum is used to detect data corruptions
and that checksum is applied to the COBS encoded data,
the checksum will be able to gate the decoding process
thus generally avoiding the error cascade.

However, if the checksum is applied to the unencoded data
the checksum will not gate the decoding process.
Instead the checksum will be asked to detect the error cascade
introduced by the decoding process.
If a CRC is used as the checksum,
its error detection performance **will be compromised** by the cascade.

In general, the probability of an undetected error in COBS framed data
protected by a CRC applied to the unencoded data is:
`1/255 * 1/2^poly_degree`.

While this is low, it is many orders of magnitude worse than simply applying the same CRC to the COBS encoded data.

