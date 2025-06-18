# C3 Specification 

## Notation

The syntax is specified using Extended Backus-Naur Form (EBNF):

```
production  ::= PRODUCTION_NAME '::=' expression?
expression  ::= alternative ("|" alternative)*
alternative ::= term term*
term        ::= PRODUCTION_NAME | TOKEN | set | group | option | repetition
set         ::= '[' (range | CHAR) (rang | CHAR)* ']'
range       ::= CHAR '-' CHAR
group       ::= '(' expression ')'
option      ::= expression '?'
repetition  ::= expression '*'
```

Productions are expressions constructed from terms and the following operators, in increasing precedence:

```
|   alternation
()  grouping
?  option (0 or 1 times)
*  repetition (0 to n times)
```

Uppercase production names are used to identify lexical tokens. Non-terminals are in lower case. Lexical tokens are enclosed in single quotes ''.

The form `a..b` represents the set of characters from a through b as alternatives.

## Source code representation

A program consists of one or more _translation units_ stored in files written in the Unicode character set, stored as a sequence of bytes using the UTF-8 encoding. Except for comments and the contents of character and string literals, all input elements are formed only from the ASCII subset (U+0000 to U+007F) of Unicode.

#### Carriage return

The carriage return (U+000D) is usually treated as white space, but may be stripped from the source code prior to lexical translation.

#### Bidirectional markers

Unbalanced bidirectional markers (such as U+202D and U+202E) is not legal.

### Lexical Translations

A raw byte stream is translated into a sequence of tokens which white space and comments are discarded. The resulting input elements form the tokens that are the terminal symbols of the syntactic grammar.

The longest possible translation is used at each step, even if the result does not ultimately make a correct program while another lexical translation would.

> Example: `a--b` is translated as `a`, `--`, `b`, which does not form a grammatically correct expression, even though the tokenization `a`, `-`, `-`, `b` could form a grammatically correct expression.

### Line Terminators

The C3 compiler divides the sequence of input bytes into lines by recognizing *line terminators*

Lines are terminated by the ASCII LF character (U+000A), also known as "newline". A line termination specifies the termination of the // form of a comment.

### Comments

There are two types of regular comments:

1. `// text` a line comment. The text between `//` and line end is ignored.
2. `/* text */` block comments. The text between `/*` and `*/` is ignored. It has nesting behaviour, so for every `/*` discovered between the first `/*` and the last `*/` a corresponding `*/` must be found.

### White Space

White space is defined as the ASCII horizontal tab character (U+0009), carriage return (U+000D), space character (U+0020) and the line terminator character (U+000D).

```text
WHITESPACE      ::= [ \t\r\n]
```

### Letters and digits

```text
UC_LETTER       ::= [A-Z]
LC_LETTER       ::= [a-z]
LETTER          ::= UC_LETTER | LC_LETTER
DIGIT           ::= [0-9]
HEX_DIGIT       ::= [0-9a-fA-F]
BINARY_DIGIT    ::= [01]
OCTAL_DIGIT     ::= [0-7]
LC_LETTER_      ::= LC_LETTER | "_"
UC_LETTER_      ::= UC_LETTER | "_"
ALPHANUM        ::= LETTER | DIGIT
ALPHANUM_       ::= ALPHANUM | "_"
UC_ALPHANUM_    ::= UC_LETTER_ | DIGIT
LC_ALPHANUM_    ::= LC_LETTER_ | DIGIT
```

### Identifiers

Identifiers name program entities such as variables and types. An identifier is a sequence of one or more letters and digits. The first character in an identifier must be a letter or underscore.

C3 has three groups of identifiers: const identifiers - containing only underscore and upper-case letters, type identifiers - starting with an upper case letter followed by at least one underscore letter and regular identifiers, starting with a lower case letter.

```text
IDENTIFIER       ::= "_"* LC_LETTER ALPHANUM_*
CONST_IDENT      ::= "_"* UC_LETTER UC_ALPHANUM_*
TYPE_IDENT       ::= "_"* UC_LETTER UC_ALPHANUM_* LC_LETTER ALPHANUM_*
CT_IDENT         ::= "$" IDENTIFIER
CT_BUILTIN_CONST ::= "$$" CONST_IDENT
CT_BUILTIN_FN    ::= "$$" IDENTIFIER
CT_TYPE_IDENT    ::= "$" TYPE_IDENT
AT_IDENT         ::= "@" IDENT
AT_TYPE_IDENT    ::= "@" TYPE_IDENT
HASH_IDENT       ::= "#" IDENT
PATH_SEGMENT     ::= "_"* LC_LETTER LC_ALPHANUM_*
```

### Keywords

The following keywords are reserved and may not be used as identifiers:

```text
any        bfloat      bool
char       double      fault
float      float128    float16
ichar      int         int128
iptr       isz         long
short      typeid      uint
uint128    ulong       uptr
ushort     usz         void

alias      assert      asm
attrdef    bitstruct   break
case       catch       const
continue   default     defer
do         else        enum
extern     false       faultdef
for        foreach     foreach_r
fn         tlocal      if
inline     import      macro
module     nextcase    null
interface  return      static
struct     switch      true
try        typedef     union
var        while

$alignof   $assert     $assignable
$case      $default    $defined
$echo      $else       $embed
$endfor    $endforeach $endif
$endswitch $eval       $error     
$exec      $extnameof  $feature
$for       $foreach    $if
$include   $is_const   $nameof
$offsetof  $qnameof    $sizeof
$stringify $switch     $typefrom
$typeof    $vacount    $vatype
$vaconst   $vaarg      $vaexpr
$vasplat
```

### Operators and punctuation

The following character sequences represent operators and punctuation.

```text
&       @       ~       |       ^       :
,       /       $       .       ;       =
>       <       #       {       }       -
(       )       *       [       ]       %
>=      <=      +       +=      -=      !
?       ?:      &&      ??      &=      |=
^=      /=      ..      ==      [<      >]      
++      --      %=      !=      ||      ::      
<<      >>      !!      ->      =>      ...
<<=     >>=     +++     &&&    |||
```

### Backslash escapes

The following backslash escapes are available for characters and string literals:

```text
\0      0x00 zero value
\a      0x07 alert/bell
\b      0x08 backspace
\e      0x1B escape
\f      0x0C form feed
\n      0x0A newline
\r      0x0D carriage return
\t      0x09 horizontal tab
\v      0x0B vertical tab
\\      0x5C backslash
\'      0x27 single quote '
\"      0x22 double quote "
\x      Escapes a single byte hex value
\u      Escapes a two byte unicode hex value
\U      Escapes a four byte unicode hex value
```

## Types

Types consist of built-in types and user-defined types (enums, structs, unions, bitstructs and typedef).

### Boolean types

`bool` may have the two values `true` and `false`. It holds a single bit of information but is
stored in a `char` type.

### Integer types

The built-in integer types:

```text
char      unsigned 8-bit
ichar     signed 8-bit
ushort    unsigned 16-bit
short     signed 16-bit
uint      unsigned 32-bit
int       signed 32-bit
ulong     unsigned 64-bit
long      signed 64-bit
uint128   unsigned 128-bit
int128    singed 128-bit
```

In addition, the following type aliases exist:

```text
uptr      unsigned pointer size
iptr      signed pointer size
usz       unsigned pointer offset / object size
isz       signed pointer offset  / object size
```

### Floating point types

Built-in floating point types:

```
float16      IEEE 16-bit*
bfloat16     Brainfloat*
float        IEEE 32-bit
double       IEEE 64-bit
float128     IEEE 128-bit*
```

(\* optionally supported)

### Vector types

A vector lowers to the platform's vector types where available. A vector has a base type and a width.

```
vector_type        ::= base-type "[<" length ">]"
```

#### Vector base type

The base type of a vector must be of boolean, pointer, enum, integer or floating point type, or a distinct type wrapping one of those types.

#### Min width

The vector width must be at least 1.

#### Element access

Vector elements are accessed using `[]`. It is possible to take the address of a single element.

#### Field access syntax

It is possible to access the index 0-3 with field access syntax. 'x', 'y', 'z', 'w' corresponds to 
indices 0-3. Alternatively 'r', 'g', 'b', 'a' may be used.

#### Swizzling

It is possible to form new vectors by combining field access names of individual elements. For example
`foo.xz` constructs a new vector with the fields from the elements with index 0 and 2 from the vector "foo". There is
no restriction on ordering, and the same field may be repeated. The width of the vector is the same as the number of
elements in the swizzle. Example: `foo.xxxzzzyyy` would be a vector of width 9.

Mixing the "rgba" and "xyzw" access name sets is an error. Consequently `foo.rgz` would be invalid as "rg" is from the "rgba" set and "z" is from the "xyzw" set.

#### Swizzling assignment

A swizzled vector may be a lvalue if there is no repeat of an index. Example: `foo.zy` is a valid lvalue, but `foo.xxy` is not.

#### Alignment

Alignment of vectors are platform dependent, but is at least the alignment of its element type.

#### Vector operations

Vectors support the same arithmetics and bit operations as its underlying type, and will perform the operation element-wise. Vector operations ignore overloads on the underlying type.

Example:

```c
int[<2>] a = { 1, 3 };
int[<2>] b = { 2, 7 };

int[<2>] c = a * b;
// Equivalent to
int[<2>] c = { a[0] * b[0], a[1] * b[1] };
```

Vectors support `++` and `--` operators, which will be applied to each element. For example, given the `int` vector `int[<2>] x = { 1, 2 }`, the expression `x++` will return the vector `{ 1, 2 }` and update the vector `x` to `{ 2, 3 }`

#### Enum vector "ordinal"

Enum vectors support `.ordinal`, which will return the ordinal of all elements. Note that the `.from_ordinal` method of enums may take a vector and then return an enum vector.

#### Vector limits

Vectors may have a compiler defined maximum bit width. This will be at least as big as the largest supported SIMD vector. A typical value is 4096 bits. For the purpose of calculating max with, boolean vectors are considered to be 8 bits wide per element.

### Array types

An array has the alignment of its elements. An array must have at least one element.

### Slice types

The slice consist of a pointer, followed by an usz length, having the alignment of pointers.

### Pointer types

A pointer an address to memory.

```text
pointer_type       ::= type "*"
```

#### Pointee type

The type of the memory pointed to is the **pointee type**. It may be any runtime type. In the case of a `void*` the pointee type is unknown.

#### Deref

Dereferencing a pointer will return the value in the memory location interpreted as the **pointee type**.

### Pointer arithmetics

An `usz` or `isz` offset may be added to a pointer resulting in a new pointer of the same type. This will offset the underlying address by the offset times the pointee size. An example: the size of a `long` is 8 bytes. Adding `3` to a pointer to a long consequently increases the address by 24 (3 * 8).

#### Subscripting

Subscripting a pointer is equal to performing pointer arithmetics by adding the index, followed by a deref.
Subscripts on pointers may be negative and will never do bounds checks.

#### `iptr` and `uptr`

A pointer may be losslessly cast to an `iptr` or `uptr`. An `iptr` or `uptr` may be cast to a pointer of any type.

#### The wildcard pointer `void*`

The `void*` may implicitly cast into any other pointer type. The `void*` pointer implicitly casts into any other pointer.

A void* pointer may never be directly dereferenced or subscripted, it must first be cast to non-void pointer type.

#### Pointer arithmetic on `void*`

Performing pointer arithmetics on void* will assume that the element size is 1.

