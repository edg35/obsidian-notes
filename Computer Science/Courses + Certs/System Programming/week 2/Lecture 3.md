---
status: done
tags:
  - pointers
  - C
date: 2026-01-28
---
### Size of Data
---

- `sizeof()` : gives you the number of bytes (chars) used to store some data
	- `sizeof(int)`
	- `sizeof(char) == 1`
	- `sizeof(struct student)`
	- `sizeof(p)`: size of a variable

### Chars
---

- `include <ctype.h>`
	- `isalpha(c)`: true if c is a letter
	- `isalphanum(c)`: true for letter or number
	- `isdigit()`
	- `toupper()`
	- `tolower()`

```c
char a[100];

a = "hello"; // no good

strcpy(a, "hello"); // same order of assignment operator

strlen(a); // a = 5
sizeof(a); // a = 100

strcmp (a,b); // lexicographical
/*
	0 if equal
	negative is a comes before b
	positive if a comes after b
*/

memcpy(dst, src, count);
/*
	requires explicitly saying howw many bytes to copy
	does not need to chek for the terminator
	this is the fastest way to copy data
*/
```

### Output
---

- `printf()`: print formatted
	- one or more arguments
	- first arguments is a format string
		- may contain format specifiers `%<code>`
	- %d = decimal
	- %o = octal
	- %x = hexadecimal

```c
printf("Here is an int %d\n", 235); // prints int
printf("Here is a long int %ld\n", someLongInt);
printf("Here is a float: %f and scientific: %e\n" float, float);
```

```c
int main (int argc, char **argv) {
	for (int i = 0; i < argc, i++){
		printf("Argument %d is %s\n", i, argv[i]);
	}
	
	return EXIT_SUCCESS;
}
```

### Pointers
---

- a pointer is just an address
	- basically a number that indicates a location in memory
- all data is accessed by address
- the compiler keeps track of where variables, literals, functions are stored
- we can explicitly get the location of data
- for variable and variable-like expressions, we can use
	- `int x = 5;`
	- `&x` - evaluates to a pointer to x / the address of x

- pointers are typed
	- e.g., a pointer to an int is typed int *

- we can create pointer variables
	- `int *p; // p is a pointer variable; it contains the address of an int`
	- `p = &x; // now p stores the address of x`

- the unary * "dereferences" a pointer; it lets us talk about the object it points to
	- p = address of x
	- *p = value of c (the pointee)
	- `*p = 10; // same as x = 10`

