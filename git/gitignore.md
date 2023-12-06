---
author: whx
title: gitignore
time: 2023-11-15-Wednesday
tags:
  - git
  - tool
  - regular-expression
---
## note

>`/` ： 结尾表示目录，及不匹配文件。

>文件的父级目录被排除，使用`!`也不会再次被包含。

>`[]` ：使用正则表达式匹配字符。

> `**`：匹配多级目录。

> `git check-ignore (-v) {file-name or directory-name}` 检测文件或目录是否被ignore。

> .gitignore只能忽略`untracked files`.

solution : 
```shell
git rm -r --cache .
git add .
```

