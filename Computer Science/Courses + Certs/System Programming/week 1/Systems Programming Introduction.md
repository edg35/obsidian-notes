---
status: done
tags:
  - intro
date: 2026-01-21
---
# Introduction

- project heavy course
	- 4 projects {programming assignments}
	- Two person projects

# What is Systems Programming?

we are going to be looking at some of the lower-level software portions of your computer/ programming environments

- language run-time system
	- operating system (OS) services
		- abstractions over hardware
		- protection form other programs
		- protection from other users
		- common implementations of useful features
		- basic user interface
	- we will look at several low level interfaces that programs use to communicate with the OS

# Why Linux / Posix?

- It's popular and freely available
- It's pretty well designed

# Why C?

- Very closely connected to Unix / Linux / Posix
	- C was created to write Unix in
- Low level access to OS features 
	- "portable assembly language"
- Puts as few barriers between programmers and the hardware
- Most modern languages are inspired by C syntax

To compile with GCC:

```bash
# compile foo.c file and name output file foo instead of a.out
gcc foo.c -o foo

# compile with warnings and 1999 version of c
gcc -Wall -std=c99 -fsanitize=address foo.c -o foo

# compile partially without libraries (seperate compilation)
gcc -Wall -c foo.c

# link
gcc foo.o -o foo

# run the program
./foo
```

- How is C different from Java?
	- We compile the code directly to machine language (executable files)
	- No virtual machine
	- compiler gcc
		- the compiler translates source code to other code. (usually machine code)


