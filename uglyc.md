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

> HW8

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

> HW8

Translate JPL types to C types as follows,
where `S` is a struct name,
`T` is any JPL type, `T+` is the C
version of the JPL type `T`, and `R` is
an array rank (number of commas + 1).


| JPL type  | C type     |
|-----------|------------|
| `int`     | `int64_t`  |
| `float`   | `double`   |
| `bool`    | `bool`     |
| `void`    | `void_t`   |
| `bool[]`  | `_a1_bool` |
| `bool[,]` | `_a2_bool` |
| `T[R]`    | `aR_T+`    |
| `S`       | `S`        |


# Memory Layout and C Struct Declarations

> HW8

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

> HW8

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

> HW8

When you find a JPL variable `x`, lookup its name, which is probably something ugly like `_0`.
Output nothing to the C program.

The name will be found if the JPL variable was previously defined (which is guaranteed
by your typechecker) and was given a C name.


## Struct Literals

> HW8

To compile a struct constructor call, such as:

```
rgba{1.0, 1.0, 1.0, 1.0}
```

First compile each of the expressions (assigning C names to their outputs)
and output a C struct constructor call:

```c
rgba _4 = {_0, _1, _2, _3}
```


## Array Literals

> HW8

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

> HW8

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


## Short-Circuit Arithmetic

> HW9

The boolean operators `&&` and `||` have short-circuiting behavior.
This means that some control flow is necessary to compile them correctly.

`&&` short circuits to false when its first argument is false.
In that case, the second argument is never evaluated.
To compile an `&&`:

- Declare a new name to hold the result. _Do not output any C._
- Compile the left-hand argument,
- Assign the result of the left-hand argument to your result variable,
- Output `if (0 == lhs)`, checking whether the left-hand result is false,
- Create a jump label for the end of the `&&` expression,
- Output a `goto` for the jump label,
- Compile the right-hand argument,
- Assign the result of the right-hand argument to your result variable,
- Output the end label

`||` short circuits when its first argument is true.
To compile an `||`, follow the same steps as `&&` with one change:

- ...
- Output `if (0 != lhs)`, checking whether the left-hand result is **true**
- ...


## Indexing

> HW8

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

> HW8

> This will be harder in assembly.

To compile a function call, generate code for the arguments left-to-right
and call the function. The function will have the same name in JPL
and in C, whether it is user-defined or built in.


# Control flow

## Conditionals

> HW8

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

> HW9

Loops generate a lot of C code.

The general strategy for loops is to initialize loop state,
run the loop body,
and increment the variables **right-to-left**.
The final step of incrementing is complicated because
we sometimes need to jump to the loop body,
sometimes need to reset a variable, and sometimes need to
do nothing (when the first loop var reaches its maximum and we are done).


### Sum loop

- Declare a name for the result integer.
- Output C to declare the result, but not to assign to it.
- Generate code **and asserts** for each of the loop-bound expressions, left to right.
  - For bound `e_i`, generate code for the expression and then immediately
    generate an assert. For example, with `_3` as the value of the bound:
    ```c
    if (_3 > 0)
    goto _jump1;
    fail_assertion("non-positive loop bound");
    _jump1:;
    ```
- Initialize the result integer to `0`.
- For each loop variable _in reverse order_:
  - Declare a C name (`_0`),
  - Output C to initialize the variable to `0`.
- Generate and output a jump label (`_jump2:;`)
- Compile the loop body expression
- Update the result variable: `_res += _x`, where `_x` is the result of the loop body.
- Output `_final++` for the final loop variable.
- For each loop variable _in reverse order_ again, starting from `_final`:
  - Check `if (_var < _bound)`, comparing the variable to its bound
  - Output a `goto` to jump to the top of the loop
  - Finally, if there are additional variables remaining:
    - Output `_var = 0;` to reset the current variable
    - Output `_next++;` to increment the next variable
- The end. Output nothing else.


### Array loop

Array loops are similar to sum loops, except they must allocate
space for a result array and update array cells as the loop runs.

Steps in **bold** below are either new or modified.

- Declare a name for the result **array**.
- Output C to declare the result, but not to assign to it.
- Generate code and asserts for each of the loop-bound expressions, left to right.
  - For bound `e_i`, generate code for the expression and then immediately
    **assign the output dimension value** and
    generate an assert.
    For example, with `_3` as the value of the bound:
    ```c
    _result.d0 = _3; // NEW do this for array loops, not sum loops
    if (_3 > 0)
    goto _jump1;
    fail_assertion("non-positive loop bound");
    _jump1:;
    ```
- **Initialize the result array by computing heap space:**
  - Declare a new C name for the size (`_4`).
  - Output C to initialize to 1: `int64_t _4 = 1;`
  - For each bound value, multiply the size: `_4 *= _bound`
  - Multiply by the size of the element type: `_4 *= sizeof(T)`
  - Allocate `_result.data = jpl_alloc(_4);`
- For each loop variable _in reverse order_:
  - Declare a C name (`_0`),
  - Output C to initialize the variable to `0`.
- Generate and output a jump label (`_jump2:;`)
- Compile the loop body expression
- **Update the result variable:**
  - Compute an index into `_result` the same way that other array indexes are computed (see above).
  - Assign `_index = _x`, where `_x` is the result of the loop body.
- Output `_final++` for the final loop variable.
- For each loop variable _in reverse order_ again, starting from `_final`:
  - Check `if (_var < _bound)`, comparing the variable to its bound
  - Output a `goto` to jump to the top of the loop
  - Finally, if there are additional variables remaining:
    - Output `_var = 0;` to reset the current variable
    - Output `_next++;` to increment the next variable
- The end.



# Commands and Statements

## let

> HW8

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

> HW8

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

> HW8

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

> HW8

To compile a print command, call the runtime function `print` with the string
argument.


## read

> HW9

To compile `read image s to v`, generate a name for the result
and output C that calls the runtime function `read_image(s)`,
where `s` is the string argument to `read`.

If the variable `v` is an array lvalue, also
output code to bind its variables using array struct fields.
Use the JPL names for the variables instead of generating C names.

Example JPL:

```
read image "rumi.jpg" to arr[H, W]
```

C output:

```c
_a2_rgba _0 = read_image("rumi.jpg");
int64_t H = _0.d0;
int64_t W = _0.d1;
```


## write

> HW9

To compile a `write` command,
simply generate code for the target expression
and call the runtime function `write_image(r, s)`,
where `r` is the result of the target expression
and `s` is the string argument to `write`.


## time

> HW9

To compile a `time` command,
generate two names and two calls to `get_time()`:

- Generate a name for the "before" time,
- Call `get_time()` to get a `double` value for the before time,
- Compile the target expression,
- Generate a name for the "after" time,
- Call `get_time()` to get a `double` value for the after time,
- Call the runtime function `print_time(a - b)`, where `a` is the after time
  and `b` is the before time.


# Function Definitions

> HW9

To compile a function declaration,
translate the header to C,
compile each of the statements,
and compile the return.
If the function has no `return`,
return a `void_t` value by default.

When translating the header,
convert all JPL types to C types
but **do not** rename variables.
Use the JPL names.
For an array lvalue, like `a[H, W]`,
the C function should declare only `a`
and access the fields using `a.d0` and `a.d1`.

Example JPL function:

```
fn bp(img[H,W] : float[,],
      i : int,
      j : int)
     : float {
  return 1.0
}
```

C output:

```c
double bp(_a2_double img,
          int64_t i,
          int64_t j) {
    double _0 = 1.0;
    return _0;
}
```


## let, assert

> HW9

The steps to compile `let` and `assert` statements are exactly
the same as the steps for the commands.


## return

> HW9

To compile a return, generate code for its target expression and return the result variable.

Example JPL:

```
    return 99 + 0
```

C output:

```c
    int64_t _0 = 99;
    int64_t _1 = 0;
    int64_t _2 = _0 + _1;
    return _2;
```


