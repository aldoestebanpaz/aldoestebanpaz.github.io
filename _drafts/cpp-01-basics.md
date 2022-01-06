# C++ part 01: Getting started

## Tools

MinGW vs ???

Compilers:
clang
gcc

Linkers:
???

Static Analysis tools:
???

Diagramming tools:
???

Libraries and Frameworks:
Qt + Qt Creator

## Smallest C++ console application

Normally in the code of a console application there must exist a 'main' function that returns 'int':

TODO: check if it is correct

```cpp
int main() { return 0; }
```

## Compiling with different compilers

### Compiling with VC++

TODO

### Compiling with GCC

TODO

### Compiling with Clang

```
clang++ main.cpp
```

## Streams

C++ uses a convenient abstraction called 'streams' to perform input and output operations in sequential media such as the screen, the keyboard or a file.

A 'stream' is an entity where a program can either insert or extract characters to/from. There is no need to know details about the media associated to the stream or any of its internal specifications. All we need to know is that streams are a source/destination of characters, and that these characters are provided/accepted sequentially (i.e., one after another).

### The standard stream objects

The standard library defines a handful of stream objects that can be used to access what are considered the standard sources and destinations of characters by the environment where the program runs:

|stream | description |
| ------- | ------- |
|cin | standard input stream |
|cout | standard output stream |
|cerr | standard error (output) stream |
|clog | standard logging (output) stream |

By default, on most program environments, the stream object 'cout' (the standard output) is defined to access the screen while 'cin' (the standard input) is defined to access the keyboard.

'cerr' and 'clog' are output streams too, so they essentially work like 'cout', with the only difference being that they identify streams for specific purposes: error messages and logging; which, in many cases, in most environment setups, they actually do the exact same thing: they print on screen, although they can also be individually redirected.

### Standard output (cout) and the insertion operator (<<)

'cout' (the standard output) used together with '<<' (the insertion operator) allows to insert formatted data into the stream:

```cpp
#include <iostream>

int main()
{
    using namespace std;

    string x = "Hello";
    
    cout << "Output sentence"; // prints: Output sentence
    cout << "\n";
    cout << 120;               // prints: 120
    cout << "\n";
    cout << x;                 // prints: Hello
    cout << "\n";

    // Multiple insertion operations (<<) may be chained in a single statement
    cout << "This " << " is a " << "single C++ statement" << "\n";
    
    return 0;
}
```

### Standard input (cin) and the extraction operator (>>)

'cin' (the standard input) used together with '>>' (the extraction operator) allows to extract formatted input data into a variable.

The extraction operator can be used on 'cin' to get strings of characters in the same way as with fundamental data types. However, 'cin' extraction always considers spaces (whitespaces, tabs, new-line...) as terminating the value being extracted, and thus extracting a string means to always extract a single word, not a phrase or an entire sentence.

```cpp
#include <iostream>

int main()
{
    using namespace std;

    int age;
    cin >> age;
    
    // extracting a string means to always extract a single word, not a phrase or an entire sentence
    string mystring;
    cin >> mystring;
    
    return 0;
}
```

### Reading lines from standard input

The standard behavior that most users expect from a console program is that each time the program queries the user for input, the user introduces the field, and then presses ENTER (or RETURN). That is to say, input is generally expected to happen in terms of lines on console programs.

To get an entire line from 'cin', a line as an input from the user, there exists a function called 'getline', that takes the stream ('cin') as first argument, and the string variable as second.

```cpp
#include <iostream>

int main()
{
    using namespace std;

    string mystr;
    cout << "What's your name? ";
    getline (cin, mystr);
    cout << "Hello " << mystr << ".\n";
    
    return 0;
}
```


References:
- [Get Started! - ISOCPP](https://isocpp.org/get-started) - A list of links to both standalone and online compilers and recommended books.
