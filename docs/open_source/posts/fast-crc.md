---
date: 2024-04-19
categories:
  - CRC
---

# Fast-CRC: _Off the shelf and on your way_

[Fast-CRC](https://github.com/leeson-consulting/fast-crc)
provides tooling for generating efficient C99 Cyclic Redundancy Check (CRC) algorithms from pick and choose templates.

Fast-CRC is provided under a permissive license (MS-PL) and may be freely used for commercial products.

Features:

- Every algorithm up to CRC-64 from Greg Cook's "CRC Catalog"
- Choice of 4-bit and 8-bit polynomial table-kernels
- Unit tested against published check values
- Dr. Nguyen's HD4 and HD6 "Fast CRC" algorithms
- Hamming profile database for every polynomial listed in Prof. Koopman's "CRC Zoo"

Roadmap:

- Support for Rust, Go, and other languages
- Support for CRC peripherals and instruction sets (eg. SSE 4.2)
- Support for T5, T6, and T7 CRC kernels
