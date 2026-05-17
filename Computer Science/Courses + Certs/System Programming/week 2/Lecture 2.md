---
tags:
  - C
status: done
date: 2026-01-26
---
### CS214 C

Why do we return int from main()?
- Exiti status
- Tells the process that invoke us whether we succeeded

**Variables**
```c
// declares a variables
// represents a location in memory that is big enough for an int
int x;

// represents the address of x
&x;

// initialize a variable when declaring it
int n = 0;

// assign to it after decalring it
int n;
n = 1;
```

- **Note**: variables in c are not initialized by default, unless they are global variables
- inside a function, variables have no value until one is put there
	- -> "indeterminate" / "random" (sort of)
### Basic Types

When we declare a variable, we have to give it a type
1.  type says how much space to reserve for the variable
2.  it also guides translation of some operation (eg. arithmetic)

What types do we have? (several flavors of int)
- int
	- regular, short, long (for regular, we write int)
	- two signed: signed, unsigned
	- c does not required specific sizes except:
		- short is at least 2 bytes
		- a long int is longer than a short int
		- int is no longer than a long int and as long as a short int
	- short -> 2 bytes
	- int -> 4 bytes
	- long -> 8 bytes
- char
	- technically, these are just small ints
- float, double
	- floating point
	- in practice IEEE 754 floating point

C will automatically insert conversions between number types as needed

```c
int x = 1;
long int y = x; // auto cast from int to long int

double z = y; // auto cast from long int to double

// for operationsm c will convert to the "largest" type
int x;
double y;

x + y; // convert x to a double and perfomr floating-point addition
int z = x + y; // convert x to a double, convert back to a int

int a = 5;
double b = a / 2

// b = 2.0 not 2.5
// to avoid:
double b = (double) a / 2;
double b = a / 2.0;

```

- No string
	- character arrays
- No Bool
	- ints
	- 1 = true
	- 0 = false

```c
if (x == 0) {...}

// same as
if (!x) {...}
```

### Arrays

- Region of memory that stores a sequence of values
- We can make array variables

```c
int v[10]; // declares v as an array of 10 ints

int u[5], q[6], t; // two arrays and a regualr int

int m[10][10]; // array of arrays of ints

v[1]; // second element of v
v[0] = 20; // writes to v

// initialize arrays
int a[3] = {10, 20, 30};

// can also leave dimentions implicit
int b = {10, 20, 30}

// char arrays
char s[] = "Hello World";

// same as
char s[] = {'H', 'e', 'l', ...., '/0'};
```

- **Note** the dimension is written with the name, not the type
	- dimensions must be constant

### Enums

- enums fancy ints
- types with a small set of possible values

```c
enum direction {up, down, left, right};

enum direction a = up;
if (a == down) {...}

switch (a) {
	case up: ...
	case down: ...
	case left: ...
	case right: ...
	case default:...
}

enum direction b = 3; // same as b = right
```

- these are much more efficient because they are ints

### Structs

- like arrays, structs are a way to bundle multiple values together
	- but they are referred to by name
	- but they can be different types

```c

struct point {double x;, double y;};

struct point p;
p.x = 1;
p.y = -2.6;

// special init
struct point p = {1, -2.6};

struct triangle {
	struct point vertex[3];
	enum color background;
	char name[255];
}

struct triangle my_triangle = {
	{{0,0}, {1,1}, {1,0}},
	blue,
	"special triangle"
}

my_triangle.vertex[0].x = 0.5;
if (my_triangle.color == blue) {...}

```

### Unions

- a struct contains all of its elements
- a union contains one of it's elements

```c
union foo {
	int i;
	double d;
}

union foo x; 
// x may contain an int or a double
// x is larger enough to store a double

x.i = 1; // now x contains an int (1)
x.d = 1; // now x contains a double (1.0)
```
