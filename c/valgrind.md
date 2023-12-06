---
author: whx
title: valgrind
time: 2023-11-17-Friday
tags:
  - c
  - debug
  - valgrind
---
## options

1. `--track-origins=yes` : track errors origin
2. `-s` : List of detected and suppressed errors
3. `--leak-check=full` : See details of leaked memory 
4. `--leak-check=full --show-leak-kinds=all` : See reachable blocks (those to which a pointer was found) in leaked memory summary.