---
aliases:
  - C预处理器
tags:
  - c
  - preprocessor
summary: 
create time: 2023-11-29T18:30:00
modify time:
---
## # and ## Operators in C

### Stringizing operator (#)

The **stringizing operator (#)** is a preprocessor operator that causes the corresponding actual argument to be enclosed in **double quotation marks.** The # operator, which is generally called the **stringize** operator, turns the argument it precedes into a **quoted string**. It is also known as the stringification operator.
It is generally used with macros in C.
#### Example

The following C code demonstrates the usage of the Stringizing operator (#).

```c
// C program to illustrate (#) operator 
#include <stdio.h> 

// Macro definition using the stringizing operator 
#define mkstr(s) #s 
int main(void) { 
	// Printing the stringized value of "geeksforgeeks" 
	printf(mkstr(geeksforgeeks)); 
	return 0; 
}
```

#### output

- $ geeksforgeeks

#### Explanation

The following preprocessor turns the line printf(mkstr(geeksforgeeks)); into printf(“geeksforgeeks”);

### Token-pasting operator (##)

The **Token-pasting operator (##)** allows tokens used as actual arguments to be concatenated to form other tokens. It is often useful to merge two tokens into one while expanding macros. This is called token pasting or token concatenation.

The ‘##’ pre-processing operator performs token pasting. When a macro is expanded, the two tokens on either side of each ‘##’ operator are combined into a single token, which then replaces the ‘##’ and the two original tokens in the macro expansion.

#### Examples

The following C code demonstrates the usage of the Token-pasting operator (##).

```c
// C program to illustrate (##) operator 
#include <stdio.h> 

// Macro definition using the Token-pasting operator 
#define concat(a, b) a##b 
int main(void) 
{ 
	int xy = 30; 

	// Printing the concatenated value of x and y 
	printf("%d", concat(x, y)); 
	return 0; 
}

```

#### output

- $ 30

#### Explanation

The preprocessor transforms printf(“%d”, concat(x, y)); into printf(“%d”, xy);

### Application of Token-pasting operator (##)

The ## provides a way to concatenate actual arguments during macro expansion. If a parameter in the replacement text is adjacent to a ##, the parameter is replaced by the actual argument, the ## and surrounding white space are removed, and the result is re-scanned.

> [# and ## Operators in C](https://www.geeksforgeeks.org/stringizing-and-token-pasting-operators-in-c/?ref=lbp)

