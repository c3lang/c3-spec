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