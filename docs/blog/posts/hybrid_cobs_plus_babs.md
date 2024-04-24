---
date: 2024-04-25
categories:
  - Checksums
  - CRC
  - Hashes
  - COBS
  - BABS
  - Byte Stuffing
  - Protocol Design
---

# Hybrid COBS + BABS (HCBS)

This post builds on what we've seen from COBS and BABS
to define a simple hybridisation of the two
that is not susceptible to error cascade
while balancing transmission and computational overhead.

## Initial COBS Link

The classical COBS chain starts with a COBS Link Byte.
This guarantees no additional overhead for the next 255 bytes.
But since data lengths are often shorter than this,
it raises the question of whether the initial COBS Link can be made smaller.

HCBS permits the Encapsulating Protocol to define the width of the initial COBS Link.
For short messages, a 5-bit or even 4-bit link may be sufficient.

Relaxing this restriction allows the initial COBS Link to be worked into the Protocol's Frame Header,
should the design accomodate.

The potential benefit is obvious - one less byte overhead for short messages.

## BABS CRC - Space Cost

As we saw earlier, the Byte Stuffing mechanism may present issues for data CRCs.
For COBS, the simple approach of Byte Stuffing the data and its CRC together can compromise error detection.
However, the alternative of separately Byte Stuffing the CRC of the COBS-data increases transmission overhead.

A solution to these problems is to:

1. Calculate the COBS-data CRC,
2. Using a lower degree polynomial than normal,
3. Then BABS the CRC,
4. And append to the COBS-data

`BABS(CRC_Degree)` will add _at most_ one bit overhead, which means that `BABS(CRC_15)` takes the same space as `CRC_16`.
Similarly, `BABS(CRC_31)` takes the same space as `CRC_32`.

Specific polynomial choices depend on the design requirements, but some _good_ lower degree polynomials include:

- `CRC-15, 0x60d` HD4 to 2046 data bytes
- `CRC-15, 0x2e75` HD6 to 14 data bytes
- `CRC-23, 0x5781eb` HD6 to 253 data bytes
- `CRC-31, 0x69f3cf97` HD6 to 4092 data bytes
- etc.

## BABS CRC - Computational Cost

Commonly used CRC polynomials require no more than 4 bytes to store the CRC.
This means that conversion from `Base_256` to `Base_255` requires at most 4 modulo-division operations (if using the standard base conversion algorithm).

On architectures supporting both integer multiplication and division this can be fast, eg. 2-12 cycles per division on ARM Cortex-M3 and above.
On architectures supporting only integer multiplication, the modulo-division can be efficiently wrapped by a function performing multiplication by a constant `1/255`, which is around [5 cycles per division](https://en.wikipedia.org/wiki/Division_algorithm).

Mileage may vary, but the overall computational cost of `BABS(CRC_X)` should be less than 30 instructions on most platforms.

## Scratching the Surface

HCBS is a simple hybridisation that offers some savings,
but the real savings come when the Encapsulating Protocol makes use of them.

I am developing one such protocol that I plan to release as FOSS.
This will take some time, but if you're interested in helping let me know.
