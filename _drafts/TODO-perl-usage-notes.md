---
title: "Perl usage notes"
tags: usage perl
---

# Perl usage notes

TODO: what are here-docs?

## Portable shebang

Instead of using the fixed '/usr/bin/perl' or '/usr/local/bin/perl' path, leave
'/usr/bin/env' to find the perl executable for you:

```perl
#!/usr/bin/env perl
```

## Always use 'use strict;'

`use strict;` is a pragma (an instruction to the Perl interpreter to do
something special when it runs your program) that enables 3 important features
in one line:

* `use strict 'vars';` generates a compile-time error if you access a variable
  that was neither explicitly declared (using any of my, our, state, or use
  vars). This helps to find typos in variable names. A common case is
  forgetting to rename an instance of a variable when cleaning up or
  refactoring code.
* `use strict 'refs';` only hard references will be allowed for the rest of the
  file. Generates a runtime error if you use symbolic references.
* `use strict 'subs';` compile-time error if you try to use a bareword
  identifier in an improper way.

Try to put this line in the top of each perl file (both
scripts and modules) always.

Reference: https://perldoc.perl.org/strict

## Use 'use warnings;' whenever possible

The warnings pragma gives control over which warnings are enabled in which parts of a Perl program. It's a more flexible alternative for both the command line flag -w and the equivalent Perl variable, $^W.

This pragma works just like the strict pragma. This means that the scope of the warning pragma is limited to the enclosing block.

By default, optional warnings are disabled, so any legacy code that doesn't attempt to control the warnings will work unchanged.

All warnings are enabled in a block by either `use warnings;` or `use warnings 'all';`.

**Default Warnings and Optional Warnings**

With the introduction of lexical warnings, mandatory warnings now become default warnings. The difference is that although the previously mandatory warnings are still enabled by default, they can then be subsequently enabled or disabled with the lexical warning pragma `no warnings;`.

Note that neither the -w flag or the $^W can be used to disable/enable default warnings. They are still mandatory in this case.

Reference: https://perldoc.perl.org/warnings

## Perl documentation

The perldoc program gives you access to all the documentation that comes with Perl with the command `perldoc <PageName|ModuleName|ProgramName|URL>`.

```sh
# Examples

# List of documentation pages organized in sections and basic info
# https://perldoc.perl.org/perl
perldoc perl

# A general intro for beginners and provides some background to help you navigate the rest of Perl's extensive documentation
# https://perldoc.perl.org/perlintro
perldoc perlintro

# To learn more things you can do with perldoc ...
# https://perldoc.perl.org/perldoc
perldoc perldoc

# For more information on perl command line options ...
# https://perldoc.perl.org/perlrun
perldoc perlrun

# Perl operators and precedence
# https://perldoc.perl.org/perlop
perldoc perlop

# Perl subroutines
# https://perldoc.perl.org/perlsub
perldoc perlsub

# Get documentation of installed perl module
# perldoc [Module]
perldoc Text::Histogram
```

The 'perldoc' command looks up documentation in .pod format that is embedded in the perl installation tree or in a perl script, and displays it using a variety of formatters. This is primarily used for the documentation for the perl library modules.

Your system may also have man pages installed for those modules, in which case you can probably just use the man(1) command.

If you are looking for a table of contents to the Perl library modules documentation, see the 'perltoc' (https://perldoc.perl.org/perltoc) page.

```sh
# Documentation of a builtin function
# -f builtinfunc
# Will extract the documentation of the perl built-in function 'builtinfunc' from perlfunc (https://perldoc.perl.org/perlfunc).
perldoc -f sprintf


# Documentation of a Perl API function
# -a perlapifunc
# Will extract the documentation of the perl api function 'perlapifunc' from perlapi (https://perldoc.perl.org/perlapi).
perldoc -a newHV

# Documentation of a FAQ questions
# -q perlfaq-search-regexp
# Will search the question headings matching the 'perlfaq-search-regexp' regular expression in perlfaq[1-9] (https://perldoc.perl.org/perlfaq, https://perldoc.perl.org/perlfaq1, ..., https://perldoc.perl.org/perlfaq9) and print these entries.
perldoc -q shuffle

# Documentation of a Perl predefined variable
# -v perlvar
# Will extract the documentation of a Perl predefined variable from perlvar (https://perldoc.perl.org/perlvar).
perldoc -v '$"'
perldoc -v @+
perldoc -v DATA
```

## Variable types and variable names

Perl has three built-in data types: scalars, arrays of scalars, and associative
arrays of scalars, known as "hashes". A scalar is a single string (of any size,
limited only by the available memory), number, or a reference to something
(which will be discussed in perlref). Normal arrays are ordered lists of scalars
indexed by number, starting with 0. Hashes are unordered collections of scalar
values indexed by their associated string key.

Scalar values are always named with '$', even when referring to a scalar that is
part of an array or a hash. The '$' symbol works semantically like the English
word "the" in that it indicates a single value is expected.

Entire arrays (and slices of arrays and hashes) are denoted by '@', which works
much as the word "these" or "those" does in English, in that it indicates
multiple values are expected.

Entire hashes are denoted by '%' and works similar to '@', indicates
multiple values.

In addition, subroutines are named with an initial '&', though this is optional
when unambiguous, just as the word "do" is often redundant in English.

Finally, symbol table entries can be named with an initial '*',

Do not confuse, for example, @\_ with $\_; $\_[0] and $\_[1] (elements of the
@\_ array) have nothing whatsover to do with $\_ (a complete distinct scalar
variable). The same idea applies for hashes.

```perl
$var = 'Hello World';

@var = 1..9;
$var[9] = 10;

%var = (
    'key1' => 'foo',
    'key2' => 'bar',
    'key3' => 'baz',
);
$var{'key4'} = 'xoxo';


print "$var\n";
# prints: Hello World
print "@var\n";
# prints: 1 2 3 4 5 6 7 8 9 10

# neither print "%hash" nor print %hash works.
# This is an interpolation trick which treats the hash as a list.
# NOTE: You cannot predict or control the output order of the key-value pairs here.
print "@{[ %var ]}\n";
# printed: key2 bar key3 baz key4 xoxo key1 foo
```

Reference:
https://perldoc.perl.org/perlintro#Perl-variable-types
https://perldoc.perl.org/perldata#Variable-names

## Expressions and the context

In Perl, it's the operator that decides what is the result, not the
values; e.g. 2*3 returns the number 8, while 2x3 returns the string '222'.

A given expression may mean a different thing depending upon where it appears and
'how you use it'. The context refers to how you use an expression.

Example of some common contexts:

```
$scalar_var                   = some_expression;    # scalar context

$array_var[some_expression]   = some_expression;    # scalar context

if(some_expression) { ... }                         # scalar context

while(some_expression) { ... }                      # scalar context


@array_var                    = some_expression;    # list context

($scalar_var)                 = some_expression;    # list assignment => list context

push @array_var, some_expression;                   # list context

print some_expression;                              # list context

foreach $scalar_var (some_expression) { ... }       # list context
```

There are two major contexts: list context and scalar context; this means that
commonly an expression can produce a list or a scalar, depending upon context.

Certain operations return list values in contexts wanting a list, and scalar
values otherwise. If this is true of an operation it will be mentioned in the
documentation for that operation. In other words, Perl overloads certain
operations based on whether the expected return value is singular or plural.

In a reciprocal fashion, an operation provides either a scalar or a list context
to each of its arguments.

The following code shows an example of using expressions, that are typically
used in a scalar context, in a list context. In this scenario, if an expression
doesn't normally have a list value, the scalar value is automatically promoted
to make a one-element list:

```perl
# list context: 2 * 2 => 4 => (4)
@array = 2 * 2;


# list context: "foo" . "bar" => "foobar" => ("foobar")
@array = "foo" . "bar";


# list context: undef => (undef) is an array of one element, NOT an empty array
@array = undef;


# list context: () is an empty array
@array = ();
```

The following code shows an example of using expressions, that are typically
used in a list context, in an scalar context. This is a more complex scenario
because not always an expression, that is typically used in a list context,
gives the number of elements in an scalar context; e.g. some expressions don't
have a scalar context value at all:

```perl
@array = ('baz', 'foo', 'bar');

# list context: @array gives the list of elements.
@array_copy = @array;
@sorted_array = sort @array;
print "@sorted_array\n";
# prints: bar baz foo


# scalar context: 'print' gives a list context, but on occasion you may need to
# force scalar context.
print "size: ", scalar @array, "\n"
# prints: size 3


# scalar context: @array gives the number of elements.
$array_size = @array;
$sum_result = $array_size + @array;
print "$sum_result\n";
# prints: 6


# scalar context: 'reverse' returns the result of concatenating the elements in
# an string and reversing it.
# Not always an expression, that is typically used in a list context, gives the
# number of elements in an scalar context.
$reversed_string = reverse ( 'Aldo', 'Paz' ); # 'AldoPaz' => 'zaPodlA'
print "$reversed_string\n";
# prints: zaPodlA


# scalar context: 'sort' returns 'undef' in an scalar context.
# some expressions don't have a scalar context value at all.
$scalar_var = sort @array;
print "undef\n" if (! defined $scarlar_var);
# prints: undef
```

Reference>
https://perldoc.perl.org/perldata#Context

## Scalars

All data in Perl is a scalar, an array of scalars, or a hash of scalars. A scalar may contain one single value in any of three different flavors: a number, a string, or a reference.

```perl
$dec_number = 1_000 + 24;        # same as 1000.
$hex_number = 0xff;
$bin_number = 0b1111_1111;       # same as 255.

# Single-Quoted string -- only \' and \\ have a special meaning.
$singlequoted_string = '\'Hello World' . '!\'';
# Double-Quoted string -- allows string interpolation, escape characters, and unicode.
# For string interpolation, curly braces are optional and need for separation in some cases.
$doublequoted_string = "\$singlequoted_string = $singlequoted_string\n";
$doublequoted_string .= $singlequoted_string . "\n";
$doublequoted_string .= "\${singlequoted_string} = ${singlequoted_string}\n";
$doublequoted_string .= ${singlequoted_string} . "\n";
$doublequoted_string .= 'Again!' . "\n";

print 'xo' x 2, "\n";                   # prints xoxo.
print "10" * '10', "\n";                # 10 * 10 = 0 prints 100.
print "foo" + 1, "\n";                  # 0 + 1 = 0, then prints 0.

# ==, !=, <, >, <=, >= for number comparison.
print "10 is greater than 1\n" if (10 > 1);
# eq, ne, lt, gt, le, ge for string comparison.
print "\"foo\" is greater than \"bar\"\n" if ("foo" gt "bar");
```

### Null strings

There are actually two varieties of null strings (sometimes referred to as "empty" strings), a defined one and an undefined one.

- The defined version is just a string of length zero, such as "".
- The undefined version is the value that indicates that there is no real value for something, such as when there was an error, or at end of file, or when you refer to an uninitialized variable or element of an array or hash.

You can use the defined() operator to determine whether a scalar value is defined (this has no meaning on arrays or hashes), and the undef() operator to produce an undefined value.

### Booleans

A scalar value is interpreted as FALSE in the Boolean sense if it is undefined (undef), the null string or the number 0 (or its string equivalent, "0"), and TRUE if it is anything else. The Boolean context is just a special kind of scalar context where no conversion to a string or a number is ever performed. Negation of a true value by ! or not returns a special false value. When evaluated as a string it is treated as "", but as a number, it is treated as 0. Most Perl operators that return true or false behave this way.

```perl
print "foo" gt "bar"; # true, prints 1
print "\n";
print 10 < 1;         # false, prints empty string
print "\n";
print !! 'foo';       # true, prints 1
print "\n";
print !! '0';         # false, prints empty string
print "\n";
print 'true' if (1 && 2 && 100 && "foo");
print "\n";
print 'false' if (! (0 || '' || '0' ));  # yes, '0' is considered a falsy value
print "\n";
```

### The 'undef' value and the 'defined' function

Variables have the special 'undef' value before they are first assigned. It acts
like 0 if used like a number or like an empty string if used line an string.

To tell whether a value is specifically 'undef' and not an empty string or 0,
use the 'defined' function, which returns false for 'undef', and true for
everything else.

```perl
for ($i = 2; $i <= 4; $i += 2) {
    # 1st iteration: 0 + 2 = 2
    # 2nd iteration: 2 + 4 = 6
    $sum += $i;
    print "$sum\n";
}
print "$sum\n";   # 2 + 4 = 6, prints 6


$string .= "more text\n";   # similar to: $string = "" . "more text\n";
print $string     # prints 'more text' plus a newline


# The following lines prints 'No defined' plus a newline
$nodefined = undef;
print "No defined\n" if (! $nodefined);
print "No defined\n" if (! defined $nodefined);
```

### Quote and quote-like operators

While we usually think of quotes as literal values, in Perl they function as
operators, providing various kinds of interpolating and pattern matching
capabilities. Perl provides customary quote characters for these behaviors, but
also provides a way for you to choose your quote character for any of them.

```
Customary  Generic        Meaning        Interpolates
    ''       q{}          Literal             no
    ""      qq{}          Literal             yes
    ``      qx{}          Command             yes*
            qw{}         Word list            no
    //       m{}       Pattern match          yes*
            qr{}          Pattern             yes*
             s{}{}      Substitution          yes*
            tr{}{}    Transliteration         no (but see below)
             y{}{}    Transliteration         no (but see below)
    <<EOF                 here-doc            yes*

    * unless the delimiter is ''.
```

Example:

```perl
print q{foo{bar}baz};  # same as: print 'foo{bar}baz'
# prints
#       foo{bar}baz

$samplevar = 123;
print qq/foo $samplevar\n/;  # same as: print "foo $samplevar\n"
```

Reference:
https://perldoc.perl.org/perlop#Quote-and-Quote-like-Operators

## Lists and arrays

A list is a sequence of scalar values (e.g. `(1, 2, 3)`). An array is a variable
that contains the list.

```perl
$array1[0] = "foo";
$array1[1] = "bar";
$array1[4] = "baz";             # there are 2 undef elements (index 2 and 3)
print "$array1[1 - 1]\n";    # same as: print "$array1[0]\n";
print "$array1[$#array1]\n"; # same as: print "$array1[-1]\n";  --  same as: print $array1[9];
print "$array1[-5]\n";      # same as: print "$array1[-5]\n";


@array2 = ();                   # empty list
@array2 = (1..3, undef, 'Hello', @array1);
foreach $item (@array2) {
    # NOTE: the control variable is not a copy, it IS the list element
    # so if you modify it, you will be changing the value of the element in the list
    $item = "undef" if !defined($item)
}
print "@array2\n";
# prints: 1 2 3 undef Hello foo bar undef undef baz


foreach (@array2) {
    print $_ eq "undef" ? "(undef)" : "<$_>";
}
print "\n";
# prints: <1><2><3>(undef)<Hello><foo><bar>(undef)(undef)<baz>


# 'qw' is a quote-like operator for list creation.
# 'qw' stands for "quoted words" or "quoted by whitespace".
# 'qw' makes easy the generation of lists of simple words,
# whitespeace is ignored and each word is interpreted like a single-quoted
# string, so you can't use escape characters or variable interpolation.
@array3 = qw! Hi
    my name is Aldo\! !;
print "$_\n" foreach @array3;  # same as: print "$_\n" for @array3;
# prints:
#       Hi
#       my
#       name
#       is
#       Aldo!


$array1 = @array1;
print "Size array1: $array1\n";          # same as: print "Size array1: " . (0 + @array1) . "\n";
print 'Size array2: ' . (scalar @array2) . "\n";
print "Size array3: " . ($#array3 + 1) . "\n";
# prints:
#       Size array1: 5
#       Size array2: 10
#       Size array3: 5
```

### Array interpolation

Keep in mind that perl gives a special meaning to the at character on
double-quoted strings, for example, when writing an email address:

```perl
@array = 1..5;
print "\@array values: @array\n";
# prints: @array values: 1 2 3 4 5
print "\$array[4] value: $array[5 - 1]\n";
# prints: $array[4] value: 5


# this is not my email :)
$email = "aldoesteban\@gmail.com";
print $email;  # prints: aldoesteban@gmail.com
```

### each function and foreach with indices

```perl
@array = 1..10;
while(my($index, $value) = each @array) {
    print "<$index:$value>";
}
print "\n";
# prints: <0:1><1:2><2:3><3:4><4:5><5:6><6:7><7:8><8:9><9:10>


# similar to ...


foreach(0..$#array) {
    print "<$_:$array[$_]>";
}
print "\n";
# prints: <0:1><1:2><2:3><3:4><4:5><5:6><6:7><7:8><8:9><9:10>
```

### List assignment

```perl
@array = qw# Hello World #;
($var1, $var2) = @array;

print "$var1 $var2!\n";
# prints:
#       Hello World!

# userful for making swap of values
($var1, $var2) = ($var2, $var1);
print "$var1 $var2!\n";
# prints:
#       World! Hello
```

### The pop, push, shift and unshift functions

```perl
@array = 1..10;

$item = pop @array;  # gets 10
print "Item value: $item  --  Array size: " . (scalar @array) . "\n";
# prints: Item value: 10  --  Array size: 9

push @array, $item;  # puts 10 in the end
print "Item value: $item  --  Array size: " . (scalar @array) . "\n";
# prints: Item value: 10  --  Array size: 10

push @array, qw^ foo bar ^;  # puts 'foo' and 'bar' in the end
print "Item value: $item  --  Array size: " . (scalar @array) . "\n";
# prints: Item value: 10  --  Array size: 12


$item = shift @array;  # gets 1
print "Item value: $item  --  Array size: " . (scalar @array) . "\n";
# prints: Item value: 1  --  Array size: 11

unshift @array, $item;  # puts 1 in the beggining
print "Item value: $item  --  Array size: " . (scalar @array) . "\n";
# prints: Item value: 1  --  Array size: 12

unshift @array, (-2, -1, 0);  # puts -2, -1 and 0 in the beggining
print "Item value: $item  --  Array size: " . (scalar @array) . "\n";
# prints: Item value: 1  --  Array size: 15
```

### The splice function

This function allows adding and removing elements in any part of an array.

```perl
# splice ARRAY, OFFSET
@array = 1..10;
@removed = splice @array, 2; # Removes eveything starting from index 2
print "@removed\n";
# prints: 3 4 5 6 7 8 9 10

# splice ARRAY, OFFSET, LENGTH
@array = 1..10;
@removed = splice @array, 0, 5; # Removes 5 elements, from index 0
print "@removed\n";
# prints: 1 2 3 4 5

# splice ARRAY, OFFSET, LENGTH, LIST
@array = 1..10;
@removed = splice @array, 0, 2, qw/ foo bar /; # Removes 2 elements, from index 0, and replaces with 'foo' and 'bar'
print "@removed\n";
print "@array\n";
# prints
#       1 2
#       foo bar 3 4 5 6 7 8 9 10


# splice ARRAY, OFFSET, LENGTH, LIST
@array = 1..10;
@removed = splice @array, 1, 0, qw/ foo bar /; # Removes 0 elements, from index 1, and puts 'foo' and 'bar' in front
print "@removed\n";
print "@array\n";
# prints:
#
#       1 foo bar 2 3 4 5 6 7 8 9 10
```

Reference:
https://perldoc.perl.org/functions/splice

### reverse and sort functions

```perl
@reverse_array = reverse 1..10;
print "@reverse_array\n";
# prints: 10 9 8 7 6 5 4 3 2 1

# By default 'sort' sorts in standard string comparison order.
@sorted_array = sort qw$ foo bar baz $;
print "@sorted_array\n";
# prints: bar baz foo

# sort numerically ascending
# using a custom comparison block with the binary "<=>" operator that returns
# -1, 0, or 1 depending on whether the left argument is numerically less than,
# equal to, or greater than the right argument.
@sorted_array = sort {$a <=> $b} (3, 10, 1);
print "@sorted_array\n";
# prints: bar baz foo
```

## Hard references (a.k.a. just References) and Symbolic references

"Hard references" are more like hard links in a Unix file system: They are used to access an underlying object without concern for what its (other) name is.

```perl
# Reference form 1: put a \ in front of a variable
$sref = \$scalar;        # $sref now holds a reference to $scalar
$aref = \@array;         # $aref now holds a reference to @array
$href = \%hash;          # $href now holds a reference to %hash
# you can copy it or store it just the same as any other scalar value
$xy = $aref;             # $xy now holds a reference to @array
$p[3] = $href;           # $p[3] now holds a reference to %hash
$z = $p[3];              # $z now holds a reference to %hash


# deferefence form 1...
@{$aref}                 # same as @array
${$aref}[3] = 17;        # same as $array[3] = 17;
%{$href}                 # same as %hash
${$href}{'red'} = 17     # same as $hash{'red'} = 17

# NOTE 1: You can omit the curly brackets whenever the thing inside them is an atomic scalar variable like $aref.
# @$aref is the same as @{$aref}
# $$aref[1] is the same as ${$aref}[1]


# dereference form 2...

$aref->[3]               # same as ${$aref}[3], don't confuse this with $aref[3], which is the fourth element of a totally different array, one deceptively named @aref.
$href->{red}             # same as ${$href}{red}, don't confuse this with $href{'red'}, which is part of the deceptively named %href hash.

# NOTE 2: It's easy to forget to leave out the ->, and if you do, you'll get bizarre results when your program gets array and hash elements out of totally unexpected hashes and arrays that weren't the ones you wanted to use.



# Reference form 2: [ ITEMS ] makes a new, anonymous array, and returns a reference to that array. { ITEMS } makes a new, anonymous hash, and returns a reference to that hash.
$aref = [ 1, "foo", undef, 13 ];    # $aref now holds a reference to an array, same as "@array = (1, 2, 3); $aref = \@array;"
$href = { APR => 4, AUG => 8 };     # $href now holds a reference to a hash

# NOTE 3: If you write just [], you get a new, empty anonymous array. If you write just {}, you get a new, empty anonymous hash.

# NOTE 4: You might prefer to go on to perllol (https://perldoc.perl.org/perllol) instead of perlref (https://perldoc.perl.org/perlref); it discusses lists of lists and multidimensional arrays in detail. After that, you should move on to perldsc (https://perldoc.perl.org/perldsc); it's a Data Structure Cookbook that shows recipes for using and printing out arrays of hashes, hashes of arrays, and other kinds of data.
```

When the word "reference" is used without an adjective, it is usually talking about a "hard reference".

"Symbolic references" are names of variables or other objects, just as a symbolic link in a Unix filesystem contains merely the name of a file.

```perl
$name = "foo";
$$name = 1;                 # Sets $foo
${$name} = 2;               # Sets $foo
${$name x 2} = 3;           # Sets $foofoo
$name->[0] = 4;             # Sets $foo[0]
@$name = ();                # Clears @foo
&$name();                   # Calls &foo()

${ bareword };              # means $bareword.
${ "bareword" };            # symbolic reference.
```

NOTE: This is powerful, and slightly dangerous, in that it's possible to intend (with the utmost sincerity) to use a hard reference, and accidentally use a symbolic reference instead. This is one of the reasons why is recommended to disable "symbolic references" using `use strict;` or `use strict 'refs';`.

References:
https://perldoc.perl.org/perlref
https://perldoc.perl.org/perlreftut


## Slots

TODO

## I/O Operations and filehandles

For your convenience, Perl sets up a few special filehandles that are already open when you run. These include STDIN, STDOUT, STDERR, and ARGV. Since those are pre-opened, you can use them right away without having to go to the trouble of opening them yourself:

```perl
print STDOUT "Please enter something: ";

# These two lines are the same as writing: chomp($line = <STDIN>);
$line = <STDIN>;     # same as: $line = readline(STDIN);
chomp $line;         # chomp operator remove trailing newline (\n) character

print "You wrote: $line\n";
print STDOUT "Thank you!\n";

print STDERR "This is a debugging message.\n";
```

As you see from above example, STDOUT and STDERR are output handles, and STDIN and ARGV are input handles. They are in all capital letters because they are reserved to Perl, much like the @ARGV array and the %ENV hash are. Their external associations were set up by your shell.

The line-input operator ('<FILEHANDLE>') returns 'undef' when there is no more input, such as at end-of-file (EOF).

```perl
# To send EOF from stdin, hit Ctrl-D in a Unix-like system, or Ctrl-Z in Windows
while (defined($line = <STDIN>)) {
    print "You wrote: $line\n";
}

print "EOF reached\n";
```

If a <FILEHANDLE> is used in a context that is looking for a list, a list
comprising all input lines (up to the EOF) is returned, one line per list
element. It's easy to grow to a rather large data space this way, so use with
care.

Each element will be an string that ends with a newline, fortunatelly 'chomp'
accept a list expression and it will remove the newlines from each item:

```perl
# These two lines are the same as writing: chomp(@lines = <STDIN>);
@lines = <STDIN>;     # same as: $line = readline(STDIN);
chomp @lines;         # chomp operator remove trailing newline (\n) character
```

References:
https://perldoc.perl.org/perlopentut
https://perldoc.perl.org/perlop

### The diamond operator <>

The null filehandle <> (sometimes called the diamond operator) is special: it can be used to emulate the behavior of sed and awk, and any other Unix filter program that takes a list of filenames, doing the same to each line of input from all of them. Input from <> comes either from standard input, or from each file listed on the command line. Here's how it works: the first time <> is evaluated, the @ARGV array is checked, and if it is empty, $ARGV[0] is set to "-", which when opened gives you standard input. The @ARGV array is then processed as a list of filenames.

```perl
# usage 1: ./script file1 [file2 ...]
#     This reads each line of each file listed as a command parameter
# usage 2: ./script
#     This read each line from STDIN (each line you type using your keyboard)
while (<>) {
    print $_;
}

# is equivalent to the following Perl-like pseudo code:
#  unshift(@ARGV, '-') unless @ARGV;
#  while ($ARGV = shift) {
#      open(ARGV, $ARGV);
#      while (<ARGV>) {
#          print $_;
#      }
#  }
```

'<>' is just a synonym for '<ARGV>', which is magical. (The pseudo code above doesn't work because it treats '<ARGV>' as non-magical.)
