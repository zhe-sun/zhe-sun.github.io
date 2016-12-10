---
layout: post
title: C macro ## ... __VA_ARGS__
description: "C macro ## ... and __VA_ARGS__"
comments: true
reading_time: true
modified: 2016-12-10
tags: [C/C++]
image:
  feature: abstract-8.jpg

---

## C macro `##` , `...` and `__VA_ARGS__`

### 1. Preprocessor Glue: The `##` Operator

预处理连接符：`##` 操作符

Like the `#` operator, the `##` operator can be used in the replacement section of a function-like macro. Additionally, it can be used in the replacement section of an object-like macro. The `##` operator combines two tokens into a single token. 

`##` 将两个符号连接成一个。
For example, you could do this:
```
#define XNAME(n) x ## n
```
Then the macro:
```
XNAME(4)
```
would expand to the following:
```
x4
```
Listing 1 uses this and another macro using `##` to do a bit of token gluing.
// `glue.c` -- use the `##` operator
```
#include <stdio.h>
#define XNAME(n) x ## n
#define PRINT_XN(n) printf("x" #n " = %d\n", x ## n);

int main(void)
{
	int XNAME(1) = 14; // becomes int x1 = 14;
	int XNAME(2) = 20; // becomes int x2 = 20;

	PRINT_XN(1);        // becomes printf("x1 = %d\n", x1);
	PRINT_XN(2);        // becomes printf("x2 = %d\n", x2);
    return 0;
}
```
Here's the output:
```
x1 = 14
x2 = 20
```
>**Note:** how the `PRINT_XN()` macro uses the `#` operator to combine strings and the `##` operator to combine tokens into a new identifier.

###2.Variadic Macros: `...` and `__VA_ARGS__`

Some functions, such as `printf()`, accept a variable number of arguments. The `stdvar.h` header file, provides tools for creating user-defined functions with a variable number of arguments. And C99 does the same thing for macros. 

Although not used in the standard, the word variadic has come into currency to label this facility. (However, the process that has added stringizing and variadic to the C vocabulary has not yet led to labeling functions or macros with a fixed number of arguments as fixadic functions and normadic macros.)

The idea is that the final argument in an argument list for a macro definition can be ellipses (that is, three periods)（`...`）. If so, the predefined macro `__VA_ARGS__` can be used in the substitution part to indicate what will be substituted for the ellipses. For example, consider this definition:
```
#define PR(...) printf(__VA_ARGS__)
```
Suppose you later invoke the macro like this:
```
PR("Howdy");
PR("weight = %d, shipping = $%.2f\n", wt, sp);
```
For the first invocation, `__VA_ARGS__` expands to one argument:
```
"Howdy"
```
For the second invocation, it expands to three arguments:
```
"weight = %d, shipping = $%.2f\n", wt, sp
```
Thus, the resulting code is this:
```
printf("Howdy");
printf("weight = %d, shipping = $%.2f\n", wt, sp);
```
Listing 2 shows a slightly more ambitious example that uses string concatenation and the # operator:

// `variadic.c` -- variadic macros
```
#include <stdio.h>
#include <math.h>

#define PR(X, ...) printf("Message" #X ": " __VA_ARGS__)
int main(void)
{
    double x = 48;
    double y;
    y = sqrt(x);
    PR(1, "x = %g\n", x);
    PR(2, "x = %.2f, y = %.4f\n", x, y);
    return 0;
}
```
In the first macro call, `X` has the value 1, so `#X` becomes `"1"`. That makes the expansion look like this:

（#为参数加双引号。）
```
print("Message " "1" ": " "x = %g\n", x);
```
Then the four strings are concatenated, reducing the call to this:
```
print("Message 1: x = %g\n", x);
```
Here's the output:
```
Message 1: x = 48
Message 2: x = 48.00, y = 6.9282
```
Don't forget, the ellipses have to be the last macro argument:
```
#define WRONG(X, ..., Y) #X #_ _VA_ARGS_ _ #y（这个是错误的例子。）
```


