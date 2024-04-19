---
date: 2024-04-19
categories:
  - CRC
  - FOSS
---

# Fast-CRC: _Off the shelf and on your way_

Fast-CRC 1.0.0 has been released on [Github](https://github.com/leeson-consulting/fast-crc/releases/tag/1.0.0) under a permissive license (MS-PL).

Fast-CRC provides tooling for generating efficient C99 Cyclic Redundancy Check (CRC) algorithms from pick and choose templates.

Features:

- Every algorithm up to CRC-64 from Greg Cook's "CRC Catalog"
- Choice of 4-bit and 8-bit polynomial table-kernels
- Unit tested against published check values
- Dr. Nguyen's HD4 and HD6 "Fast CRC" algorithms
- Hamming profile database for every polynomial listed in Prof. Koopman's "CRC Zoo"

This release has been tagged as **NOT Production Ready**.
I'm looking for guinea-pigs to test the library --
especically on Big-Endian architectures where it has had no testing to date due to a lack of hardware.
