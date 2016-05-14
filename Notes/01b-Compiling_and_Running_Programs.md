# Compiling and Running Programs

### An Example Program

Here's an example program that calculates the area of a circle. Let's suppose you've written this program in a text
editor and saved it as `area.c`.

```
// Calculate the area of a circle
// DSM, 2016

#include <stdio.h>

#define APPROX_PI 3.14159

double area(double radius) {
  return radius * radius * APPROX_PI;
}

int main() {
  double r = 5.0;
  printf("area(%f) = %f\n", r, area(r));
}
```

### Compiling with GCC

`area.c` is a source code file, but C source code means nothing to the CPU: it's just text. In order to run the program, we need to 
*compile* `area.c` into a low-level machine language program.

GCC (the GNU Compiler Collection) is a popular free software tool created by the GNU Project. I'll usually refer to GCC as "the 
compiler," but it's actually a suite of programs that do the work of transforming an input source code file into an output executable.
The compilation process has four main phases:

  1. The C preprocessor, `cpp`, which scans the input and deals with statements like `#include` and `#define`. The output of this     
     phase is another C source code file with some of the text transformed.

  2. The compiler proper, `cc1`, which takes the preprocessed source code file as input and produces an *assembly language* program
     containing the actual sequence of low-level machine instructions that make up the program in a (more or less) human-readable form. The compiler does a lot of work, including examining the syntax and structure of the source file to ensure that it's a 
     valid C program. The compiler may also *automatically optimize* the program before generating its final output.

  3. The assembler, `as`, which takes the assembly language program produced in step (2) as its input and produces a binary *object 
     file* as its output. The object file contains the same instructions as the assembly language file, but encoded in a binary format the CPU can understand.

  4. The linker, `ld`, which takes the object file produced in step (3) and converts it into an actual executable program. If 
     necessary, the linker also handles the work of combining multiple object files into one program or incorporating code from pre-compiled libraries into the final executable.
  
To compile, use `gcc` at the terminal command prompt and specify the source file. This produces an executable file named, by default, `a.out`. 

```
prompt$ gcc area.c
```

Type `./a.out` to run the program. Here, the leading `.` refers to the current directory; the command specifies that you want to run the program named `a.out` that resides in the current directory.

```
prompt$ ./a.out
area(5.000000) = 78.539750
```

Why do you have to type `./a.out` instead of just `a.out`? 

Well, a common hacker trick is to insert a malicious program into a directory and give it the same name as a popular utility like 
`ls`. If the `./` was not required, a sysadmin could type 

```
prompt$ ls
```

to list the files in the directory, but the hacker's fake `ls` would run instead of the real `ls` tool. Pwned!

Of course, only naming executables `a.out` is kind of limiting. Use the `-o` flag to specify the name of the executable. In many cases 
this will have the same name as the `.c` source file but that isn't required.

```
prompt$ gcc -o area area.c
prompt$ ./area
area(5.000000) = 78.539750
```

Make sure to put the executable name right after the `-o` flag. Running `gcc -o area.c area` will just give you an error.

### Errors and Warnings

I recommend compiling your programs with the `-Wall` flag, which gives more useful warnings about potential problems in your code.

```
prompt$ gcc -Wall -o area area.c
area.c: In function ‘main’:
area.c:15:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
```

**Fix your warnings**. The issues they identify are potential problems with your code. Better yet, use the
`-Werror` flag to force GCC to treat all warnings as errors. Remember how we said C programmers are paranoid?

So what's the problem here? 

First, read the compiler's message carefully: it gives the function and the line number containing the error.
Line 15, it turns out, is the final line of the program with the closing `}` of `main`. The final line of the warning message is
indicating that there's something wrong with that character.

Next, think about what the error message is telling you. Perhaps `non-void function` has something to do with the return type of 
`main`? Ah! `main` has a return type of `int`, but the program doesn't actually *return* an `int`: it just ends. Adding a `return` 
statement to the end of `main` will fix the warning.

```
int main() {
  double r = 5.0;
  printf("area(%f) = %f\n", r, area(r));
  
  return 0;  // My work here is done.
}
```

Technical note: some compilers (like the version of GCC on my Mac) will not generate a warning if `main` lacks a return statement. The
ANSI C99 standard specifies that reaching the end of `main` automatically returns `0`. Therefore, whether or not you see the warning 
depends on what standard your compiler has been set to follow. GCC has loads of flag options for controlling standards and 
portability—we won't use them in this class.


### Makefiles

Serious programmers (of which you are one) don't build projects manually. A large application, after all, might be built from hundreds
of source files, each with its own set of headers, libraries, and dependencies.

The standard automated build tool on Linux systems is called `make`.

```
prompt$ make
```

When you run  `make`, it looks for a file named `Makefile` (capital M, no extension) in the current directory. The Makefile contains a
set of rules that `make` uses to build your project. It will also automatically accept `makefile`, or you can identify a file of your
choice with a flag.

When I was a student, we used to say that no one had ever written a Makefile from scratch. There was just one original ur-Makefile 
carved on stone tablets by, like, the Sumerians, and then handed down from generation to generation.

A Makefile contains a series of *rules*, each of which specifies a series of build commands. The default rule is called `all`, so
running `make` with no additional arguments is equivalent to 

```
prompt$ make all
```

Suppose you've written a homework project with three source files named `hw1.c`, `hw2.c`, and `hw3.c`. You want to compile each source file into its own executable. Here's a Makefile that accomplishes this task:

```
all:
    gcc -Wall -Werror -o hw1 hw1.c
    gcc -Wall -Werror -o hw2 hw2.c
    gcc -Wall -Werror -o hw3 hw3.c
    
clean:
    rm hw1
    rm hw2
    rm hw3
```

That's it. When you run `make`, the system will execute each command. Each command is indented by *a single tab* &mdash *not spaces*. Also notice the colon after `all`.

The `clean` target removes the executables when you run

```
prompt$ make clean
```

Please include a `clean` target in any Makefile you create.

**Fancier Makefiles**



