---
date: 2024-04-24
categories:
  - CRC
  - COBS
  - Byte Stuffing
---

# Base Adaptation Byte Stuffing (BABS)

We've looked at two methods of Byte Stuffing framed data, ie. HDLC and COBS.
HDLC is fairly industrial.
It gets the job done in a simple if not efficient way.
COBS is a step up.
It makes use of the structure of the underlying data to limit overhead in a consistent way.
But there are other methods.

The _best_ I have found is called [Base Adaptation Byte Stuffing](https://web.fe.up.pt/~jsc/publications/conferences/2007JaimeICC.pdf) (BABS).
BABS is a first principles solution to the underlying problem developed by Jaime Cardoso of Universidade do Porto.

If we treat the data is an enormous big-endian number in `Base_256`,
then each digit of that number corresponds to each byte in the data.

Now, just like any number we can convert data to a different base.
Jaime chose to use `Base_255`, which of course has _one less value_ than `Base_256`.

Joining the dots we can see the big picture:

>    Converting byte-oriented data to `Base_255` can be used to eliminate the Frame Delimeter from the data.

!!! Customisation

    Frame Delimeters aren't restricted to the value `255`, so how do you deal with them?

    The default COBS Terminator `0x00` can be eliminated from the data by first performing the `Base_255` conversion,
    then `XOR`ing each byte with `0xff`.
    Other values can be elimated by transposition etc.

The conversion process expands the number of bits required to store the new number, so what's the transmission cost?
Going back to high school maths we can say that a number A requiring N `Base_256` digits
can be written as A' using M `Base_255` digits where:

`M = N * ceil(log_10(256) / log_10(255));`

Since the multiplier is constant and the expression is linear, we can precompute M for any N.
BABS turns out to be very efficient, incuring 1 byte of overhead for every 1415 data bytes!
This is close to the theorectical limit set by information theory - so we know we're done.

## Byte Stuffing Overhead - A Quick Comparison

BABS overhead is completely data agnostic - given the (usual) restriction of sending whole bytes.
This in turn means that the _worst case_ performance is the same as the _best case_ performance.

COBS overhead is slightly data gnostic.
Data containing frequent `0x00` values will have a very low overhead - as low as one byte irrespective of length.
But COBS _worst case_ overhead is consistent, ie. 1 byte every 255 data bytes.

HDLC overhead is very data gnostic.
It presents no overhead if the data is devoid of `0x7e`,
and 100% overhead if the data is competelt `0x7e`.
The average performance levelling out to around twice that of the worst COBS performance.

## What's the Cost?

MIPS.

Jaime made it clear that the algorithm used to convert `Base_256` data to `Base_255` data is `O(n^2)`.
He suggested using a simple COBS / BABS hybrid approach for framing.
When the data length was less than some computationally feasible bound - use BABS.
Otherwise fall back to COBS, which is pretty efficient for moderate sized data anyway.

We're going to follow this lead in the next post which looks at COBS + BABS hybridisation.
