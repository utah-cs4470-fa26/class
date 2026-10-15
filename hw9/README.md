Assignment 9: More C
====================

Compile the full JPL language to Ugly C.
 
```diff
cmd : show <expr>
    | print <string>
    | let <lvalue> = <expr>
    | assert <expr> , <string>
    | struct <variable> { ; <variable>: <type> ; ... ; }
+   | read image <string> to <lvalue>
+   | write image <expr> to <string>
+   | time <cmd>
+   | fn <variable> ( <binding> , ... ) : <type> { ;
+         <stmt> ; ... ;
+     }

+stmt : let <lvalue> = <expr>
+     | assert <expr> , <string>
+     | return <expr>

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

+expr : <expr> && <expr>
+     | <expr> || <expr>
+     | array [ <variable> : <expr> , ... ] <expr>
+     | sum [ <variable> : <expr> , ... ] <expr>
```

As with HW8, your C code does not need to run. The autograder will compare `.c`
files directly.

See [../hw8/running.md](../hw8/running.md) for instructions on how to actually run the C
code. Ask for help on Discord.


#### Extra Credit

The staff compiler outputs C with scoping errors. Variables that are defined at the top level
should be visible inside JPL functions, but they are not.

#### Example Program ([hw9/073.jpl](https://github.com/utah-cs4470-fa26/grader/blob/main/hw9/ok/073.jpl)):

```
read image "a.png" to x[W, H]
fn f(d : int) : int {
   return W * H + d // Should be able to use the array defined above!
}
show f(13)
```

#### C output:

```

int64_t f(int64_t d) {
    int64_t _0 = W * H; // OOPS! Want to use W but not in scope yet.
    int64_t _1 = _0 + d;
    return _1;
}

void jpl_main(struct args args) {
    _a2_rgba _0 = read_image("a.png"); // Array defined here, oops!
    int64_t W = _0.d0;
    int64_t H = _0.d1;
    int64_t _1 = 13;
    int64_t _2 = f(_1);
    show("(IntType)", &_2);
}
```

For extra credit, **after you have a full-credit solution to HW9**,
write a small patch that fixes the scope error.
Meet with an instructor at office hours or by appointment to explain your
solution.

The reward is a 0.5% increase (one half of 1 percent) added to your overall
final grade.

Everyone can get the extra credit. I just don't think it is worth your time
to do it, which is why the reward is small. But maybe you'll find a clever
solution that we can use in future years.


# Testing your code

We will run your compiler this way:

    make run TEST=/grader/ok/001.jpl FLAGS=-i

The grader directory `ok` (Part 1) contains valid hand-generated JPL programs.
The directory `ok-fuzzer` contains valid fuzzer-generated programs.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw9


# Submission and grading

This assignment is due Friday October 30. Happy Halloween.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 90%    | Part 1   |
| 10%    | Part 2   |

