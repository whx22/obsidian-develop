---
author: whx
title: file position indicator
time: 2023-11-19-Sunday
tags:
  - c
---
##  Introduction

The _file position_ of a stream describes where in the file the stream is currently reading or writing. I/O on the stream advances the file position through the file. On GNU systems, the file position is represented as an integer, which counts the number of bytes from the beginning of the file. See [File Position](https://www.gnu.org/software/libc/manual/html_node/File-Position.html).

During I/O to an ordinary disk file, you can change the file position whenever you wish, so as to read or write any portion of the file. Some other kinds of files may also permit this. Files which support changing the file position are sometimes referred to as _random-access_ files.

### fread() and fwrite() change the file position indicator

The file position indicator for the stream is advanced by the number of bytes successfully read or written.
### Reference

[File Positioning](https://www.gnu.org/software/libc/manual/html_node/File-Positioning.html)

[fsetpos() (Set File Position) in C](https://www.geeksforgeeks.org/fsetpos-set-file-position-in-c/)
