Ugly C Handbook for JPL
---

This document explains how your compiler should generate ugly C code.


# Overview

The purpose of generating C is to build intuition for generating assembly.
We're _not_ generating fast or idiomatic C code. We're generating C that
looks like the kind of assembly we'll generate in the next assigments.
That's why it's called Ugly C.

* Expressions generate code in a stack-machine style, ending by storing
  their result.
* Most control flow is via jumps, except for function calls.


## Code Structure

Ugly C programs are made of 3 sections: Header, Structs, Functions.

### Header

The **Header** is the same no matter what JPL code we are compiling:

```c
#include <math.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>
#include "rt/runtime.h"

typedef struct { } void_t;
```


### Structs

For every JPL struct and every JPL array that appears in the source code,
generate a C struct definition.

JPL structs use the same name in C. For example, a JPL struct named `three`
generates a C struct named `three`.

JPL arrays should have a name that describes their rank and element type.
A 2D array of floats has the C name `_a2_double`. The general format is `_aRANK_TYPE`.

Print the structs in topological, alphabetical order. Group the structs by their
dependencies on other structs (no deps first, 1-deps second, ...) and sort structs
alphabetically (by ASCII code) within each group.



### Functions

For every JPL function, generate a C function.

Finally, generate a function named `jpl_main` with the generated code
from all top-level commands:

```
void jpl_main(_a1_int64_t args) {
  // generate code in here
}

```


## Types

Translate JPL types to C types as follows,
where `S` is a struct name,
`T` is any JPL type, `T+` is the C
version of the JPL type `T`, and `R` is
an array rank (number of commas + 1).


| JPL type  | C type    |
|-----------|-----------|
| `int`     | `int64_t` |
| `float`   | `double`  |
| `bool`    | `bool`    |
| `void`    | `void_t`  |
| `bool[]`  | `a1_bool` |
| `bool[,]` | `a2_bool` |
| `T[R]`    | `aR_T+`   |
| `S`       | `S`       |
|-----------|-----------|


# Memory Layout and C Struct Declarations

Array elements will be stored on the heap. Use the runtime function
`jpl_alloc` to get heap space.

Everything else will be live on the stack, in local C variables:
- Integer, Float, Boolean, and Void literals
- Array structs
- JPL structs

For every JPL array, declare a struct in C. The struct must have a pointer
to its data on the heap, and dimension length fields for each dimension.

Example: the JPL type `float[,]` calls for the following C struct:

```c
typedef struct {
  int64_t d1;
  int64_t d2;
  double* data;
} _a2_double;
```

Your compiler must output a struct with exactly the same name and field names.

For every JPL struct, output a C struct with similar fields.

Example JPL struct:

```
struct three {
  b: int
  c: int
  d: int
}
```

C version:

```c
typedef struct {
    int64_t b;
    int64_t c;
    int64_t d;
} three;
```


# Instructions

Keep a **global** counter for jump targets.
Start the count at 1 and generate `_jump1`, `_jump2`, ...

Keep global lists (or dicts, etc) of all structs and all functions encountered
in the JPL code.

As you generate code for each function (including `jpl_main`),
keep a local counter for variable names. Declare variables
starting from `_0`, `_1`, ...

To begin generating C code, start by declaring a function `jpl_main`
and process the source code for all toplevel commands inside
that function.

Send information upward in the following cases:

- If you see a struct declaration, send the struct info to the global struct
  collection.
- If you see a function declaration, send it to the function collection
  and generate code for its body in that context (with a new counter for
  variable names, etc.).
- If, when processing an expression, you find an array type that hasn't been
  seen before, send an appropriate struct declaration to the global collection.

Note that this strategy produces broken C code when a toplevel command declares
a variable (`let` or `read`) and a function tries to reference that variable.


## Basic Literals

To compile an integer literal, output a C declaration such as:

```c
int64_t _0 = 9001;
```

To compile a float literal, truncate the JPL float and declare a C variable.
For the float 2.7111, the expected output is:

```c
double _1 = 2.0;
```

To compile a boolean or void constant, make a similar declaration.


## Variables

When you find a JPL variable `x`, lookup its name and output a reference:

```c
_0
```

This will work if the JPL variable was previously defined (which is guaranteed
by your typechecker) and was given a C name.


## Struct Literals

To compile a struct constructor call, such as:

```
rgba{1.0, 1.0, 1.0, 1.0}
```

First compile each of the expressions (assigning C names to their outputs)
and output a C struct constructor call:

```
rgba _4 = {_0, _1, _2, _3}
```


## Array Literals

To compile an inline array:

```
[1, 2, 3]
```

- Create a global struct declaration for the array type, if needed
- Compile each of the expressions (assigning names)
- Declare the array in C
- Set the length of the array (field `d0`)
- Allocate space for the data (size of element type * number of elements)
- Write the data

For the JPL above, the C output is:

```c
int64_t _0 = 1;
int64_t _1 = 2;
int64_t _2 = 3;
_a1_int64_t _3;
_3.d0 = 3;
_3.data = jpl_alloc(sizeof(int64_t) * 3);
_3.data[0] = _0;
_3.data[1] = _1;
_3.data[2] = _2;
```


## Arithmetic

To compile a unary operation, first compile the target expression,
then generate a name for the result and output a C declaration.

Example JPL:

```
!true
```

C:

```c
bool _0 = true;
bool _1 = !_0;
```

To compile a binary operation, first compile the left subexpression,
then compile the right subexpression, then output a C declaration
for the result.

Mod (`%`) on floats should use the `fmod` function. For other binary
operations, the JPL names and C names match.

|--------|-------|------|
| JPL op | Type  | C op |
|--------|-------|------|
| +      | N/A   | +    |
| -      | N/A   | -    |
| *      | N/A   | *    |
| /      | N/A   | /    |
| %      | int   | %    |
| %      | float | fmod |
| <      | N/A   | <    |
| >      | N/A   | >    |
| <=     | N/A   | <=   |
| >=     | N/A   | >=   |
| ==     | N/A   | ==   |
| !=     | N/A   | !=   |
|--------|-------|------|


## Indexing

### Struct Index

To compile a struct field access `e.field`, first generate code for the
struct expression (`e`), then use your typechecker results to determine
the type of C variable to declare.

For example, if the JPL variable `my_rgba` refers to an `rgba` struct,
then the field access `my_rgba.r` should output C like this, depending
on the C name for `my_rgba` and the name assigned to the result:

```c
double _3 = _0.r
```


### Array Index

To compile an array index `e1[e2]`:

- Compile the array expression `e1`,
- Compile the index value `e2`
- Output C to check whether the index is in bounds:
  - Let's assume the array is stored as `_0` and the index value is stored as
    `_1` in C.
  - First check that `_1` is non-negative:
    ```c
      if (_1 >= 0)
      goto _jump1;
      fail_assertion("negative array index");
      _jump1:;
    ```
  - Second check that `_1` is in bounds:
    ```c
      if (_1 < _0.d0)
      goto _jump2;
      fail_assertion("index too large");
      _jump2:;
    ```
- Figure out where the data is and declare a C variable for the offset.
  - Start the offset at zero `int64_t _2 = 0;`
  - Multiply by the dimension length: `_2 *= _0.d0;`
  - Add the index value: `_2 += _1;`
- Declare a name for the result of the index expression:
  - `int64_t _3 = _0.data[_2];` (or whatever the element type is)

For a multi-dimensional array, each index needs a bounds check in left-to-right
order. The offset computation (which is trivial for 1D arrays) will need to
loop over all indexes, multiplying and adding.


# Calling functions

> This will be harder in assembly.

To compile a function call, generate code for the arguments left-to-right
and call the function. The function will have the same name in JPL
and in C, whether it is user-defined or built in.


# Control flow

## Conditionals

> The C code is straightforward. The hardest part here is generating names and jumps
>  in the right order.

To compile an `if` expression, `if e1 then e2 else e3`:

- Generate code for the test expression `e1`
- Declare a new name to hold the result. Let's call it `_67`
- Create a jump label for the else branch
- Check if the result of `e1` is false, if yes then `goto` the else label
- Generate code for the "then" expression `e2`
- Assign the result of `e2` to your result name (`_67 = ...`)
- Create a jump label for the end of the if and output a `goto` for that label
- Output the else label
- Generate code for the "else" expression `e3`
- Assign the result of `e3` to your result name (`_67 = ...`)
- Output the end label

Example JPL:

```
let a = if true then 1 else 2
```

C output:

```c
bool _0 = true;
int64_t _1;
if (!_0)
goto _jump1;
int64_t _2 = 1;
_1 = _2;
goto _jump2;
_jump1:;
int64_t _3 = 2;
_1 = _3;
_jump2:;
```


## Loops

HW9


# Commands and Statements

## let

To compile a let, generate C for the right hand side
and record the generated name.

Example JPL:

```
let x = 867-5309
```

There is no C output for the `let` by itself, only for the
subtraction (`867-5309`). Your compiler must remember that
the JPL name `x` refers to a C name such as `_2`.


## assert

To compile an assert command, generate code for the target
expression and then check that the result is not zero (`0 != _999`).

Example JPL:

```
assert false, "still closed"
```

C output:

```c
bool _0 = false;
if (0 != _0)
goto _jump1;
fail_assertion("still closed");
_jump1:;
```

## show

To compile a show command, generate code for the target expression
and call the runtime function `show` with a type string and the result.

Example JPL:

```
show true
```

C:

```c
bool _0 = true;
show("(BoolType)", _0);
```

## print

To compile a print command, call the runtime function `print` with the string
argument.


# Function definitions

HW9



