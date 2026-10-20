Assignment 8: Generating C
==========================

For this assignment, compile the following subset of JPL to ugly straight-line
C code:

```
cmd : show <expr>
    | print <string>
    | let <lvalue> = <expr>
    | assert <expr> , <string>
    | struct <variable> { ; <variable>: <type> ; ... ; }

lvalue : <variable>
       | <variable> [ <variable> , ... ]

binding : <lvalue> : <type>

type : int
     | bool
     | float
     | <type> [ , ... ]
     | <variable>
     | void

expr : <integer>
     | <float>
     | true
     | false
     | <variable>
     | void

expr : - <expr>
     | ! <expr>
     | <expr> + <expr>
     | <expr> - <expr>
     | <expr> * <expr>
     | <expr> / <expr>
     | <expr> % <expr>
     | <expr> < <expr>
     | <expr> > <expr>
     | <expr> == <expr>
     | <expr> != <expr>
     | <expr> <= <expr>
     | <expr> >= <expr>

expr : <variable> { <expr> , ... }
     | [ <expr> , ... ]
     | <expr> [ <expr> , ... ]
     | <expr> . <variable>
     | if <expr> then <expr> else <expr>
     | <var> ( <expr> , ... )
```

*Your C code does not need to run.* The autograder will only check that it
matches the `.expected` files character-by-character (ignoring whitespace,
comments, etc.).

See [running.md](./running.md) for instructions on how to actually run the C
code. Ask for help on Discord.

#### Hints

* Truncate floats when printing. Example: print `3.999` as `3.0`.
* Define `void_t` in your header. It is not in the runtime library because it is an empty struct and therefore non-standard.
* There are several built-in struct types that you **must not** generate C code for.
  See [runtime.h](https://github.com/utah-cs4470-fa26/runtime/blob/main/runtime.h) for their definitions.
  - `rgba` for JPL `rgba`
  - `_a2_rgba` for JPL `rgba[,]`
  - `_a1_int64_t` for JPL `int[]` (used to pass command line arguments to the main function)
* Generate `typedef struct` code for every struct command and every array
  literal in the program.
  * Print structs in topological, alphabetical order. In more detail: build up a list of all structs
    as you process a JPL program, arrange the list into topologically-ordered groups
    and sort each group alphabetically (by ASCII code order) before printing.
* Keep a counter per generated function (including `jpl_main`) for variable names, generate names in order: `_0`, `_1`, `_2`, ...
* Convert JPL types to C types as follows:
  - `int` -> `int64_t`
  - `float` -> `double`
  - `bool` -> `bool`
  - `void` -> custom struct (see header in any `.expected` file)
  - array -> custom struct with ...
    + one int field `di` per dimension
    + one pointer field `data` for the elements
    + a name of the form `_aRANK_TYPE` where `RANK` is an integer rank and `TYPE` is the C type of its elements
    + Example: `int[,]` -> `typedef struct { int64_t d1 ; int64_t d2 ; int64_t *data ; } _a2_int64_t;`
  - struct -> struct name (which will have been defined by an earlier struct command)

Use the helper functions provided by `runtime.c` in your output, such as `jpl_alloc`.

`show` needs a type string as its first argument. For everything except structs, use the
type S-expression printer you wrote for HW6. For structs, you must **print a tuple type** with
the element types inlined.

Example:

```
struct taco {
  shell: int
  ingredients: int[]
}
show taco{2, [4,5]}
```

Output:

```
...
show("(TupleType (IntType) (ArrayType (IntType) 1))", &_6);
```

`assert` must compile to an `if` that tests for nonzero input and either falls through to an
assertion or jumps forward to safety.

Example:

```
assert 1 == 2, "stop"
```

Output:

```
...
bool _2 = _0 == _1;
if (0 != _2)
goto _jump1;
fail_assertion("stop");
_jump1:;
```


# Testing your code

We will run your compiler this way:

    make run TEST=/grader/ok/001.jpl FLAGS=-i

The grader directory `ok` (Part 1) contains valid hand-generated JPL programs.
The directory `ok-fuzzer` contains valid fuzzer-generated programs.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw8


# Submission and grading

This assignment is due Friday October 23.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 90%    | Part 1   |
| 10%    | Part 2   |

