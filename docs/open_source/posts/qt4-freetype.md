---
date: 2017-06-15
categories:
  - Qt
  - Freetype
  - Arabic
---

# Qt4-Freetype: _Backport of Freetype to Qt4 for Embedded Windows_

Qt4 is notably quicker and smaller than Qt5 on older embedded Windows platforms (Win-CE).
Unfortunately it only targets Windows native font rendering,
which lacks support for scripted forms, bidirectional input, and other advanced features.

[Freetype](https://freetype.org/) is an open source project that provides superior font rendering on many platforms.
Qt5 can target Freetype on embedded Windows platforms.

[Qt4-Freetype](https://github.com/leeson-consulting/qt/commits/4.8/)
is a backport of Qt5 Freetype support to Qt4 for embedded Windows platforms.

It has been used successfully in commercial products for the Arabic speaking market
and is free to use in production but unsupported.
