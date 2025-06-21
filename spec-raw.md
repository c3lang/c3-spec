# Raw spec discussion

The things in this spec discussion covers things not in the spec yet.

## Module search order

1. Search in local module section -> done if found
2. Search in current module -> done if found
3. Search in all imported modules
4. If there is only one match -> done
5. If there are more than one -> ambiguous match
6. Otherwise it's a failure

### Imported modules

1. All the sub-modules of the current module
2. All the imported modules and their sub-modules
3. std::core and all its sub-modules.

## Method search order

1. Search all `@public` methods for the type in all modules.
2. Search local methods in the current module section.
3. Search private method in the current module. Or any `@public` imported modules.

## Operator overloading for binary ops

### Definitions

1. Wildcard match: a match where the second parameter is untyped.
2. Direct methods: methods defined in the same module as the type

### Resolution

(Searching follows method lookup)

1. Search all lhs matches iff the lhs type is user-defined, otherwise this is a non-match.
2. Search all rhs matches iff the rhs type is user-defined, otherwise this is a non-match.
3. If either lhs or rhs resolution is ambiguous, everything is ambiguous.
4. If there is a valid lhs and rhs match, and both are non-wildcard, it is ambiguous.
5. If there is a normal match and a wildcard match, select the normal match.
6. If there are two valid wildcard matches, it is ambiguous.
7. If there is a single normal match, select the normal match.
8. If there are no matches, then the result is no match.

### Match searching

1. For lhs matching, search all methods with `@operator` and `@operator_s`, for rhs matching, search all methods with `@operator_s` and `@operator_r`
2. If the second parameter is untyped, this is a wildcard match. 
3. If the rhs type matches the second parameter type, this is a normal match.
4. If the rhs argument can be implicitly converted to the second parameter type, this is a normal match.
5. In all other cases, this is not a match.
6. If more than one normal match is found, this is an ambiguous match.
7. If one normal match is found, then this is selected over any wildcard match.
8. If no normal match is found, but multiple wildcard matches, then this is ambiguous.


### Binary expression resolution

Evaluating a binary expression passes through a series of checks and promotions.

Strictly speaking the contracted ternary `?:` and the optional-else operator `??` are binary expressions. However, their evaluation differs from the other binary expressions and so does not follow regular evaluation.

For the remaining binary expressions, an *ambiguous evaluation order check* is performed. Binary expressions are then classified into two groups: 

1. Assignment expressions: `= += -= *= /= %= ^= |= &=`
2. Non-assignment binary expressions: `+ - * / % ^ | & && || == != < <= > >= &&& ||| +++`.

#### Ambiguous evaluation order check

Three groups of operators are defined:

1. Binary bitwise operators (`& | ^`)
2. Comparisons (`== != >= <= < >`)
3. Bitshifts (`<<` and `>>`)

If the left or right hand side in itself is binary expression with the operator in the same group as the expression, then this is an error.

An exception is made for chaining of *the same* operator in group 1.

```c3
// Valid
a & b == 3
a == b << 4
a & b & c
// Invalid
a & b | c
a == b != c
a << b << c
```

### Non-assignment binary expression resolution

#### Evaluation of subexpressions
1. If the binary expression is not a logical "or" or "and", resolve lhs and rhs as values.
2. If both lhs and rhs are initializer lists, this is an *error* ⚠️.
3. If only one side is an initializer list, resolve the other side, then infer the initializer list with the type of the resolved side.
4. Otherwise, resolve the left hand side.
5. If the lhs is an enum type, resolve the other side with inferred as an enum.
6. Otherwise, resolve the rhs as a value.

#### Additional conversions

1. If the operation is `+` and the lhs is a pointer, implicitly cast rhs to usz/isz using *signed implicit binary casting*. If instead the rhs is the pointer, implicitly cast lhs in the same way.
2. If the operation is `+` and the lhs is a pointer vector, implicitly cast rhs to usz/isz vector using *signed implicit binary casting*. If instead the rhs is the pointer vector, implicitly cast lhs in the same way.
3. If the operation is `-` and the lhs is a pointer or pointer vector, and the rhs is not a pointer nor pointer vector, implicitly cast to usz/isz using *signed implicit binary casting*.

### Optional else "??"

If the expression is on the form `a ?? b`, the common binary evaluation is skipped in favour of the following:

1. Ternary ambiguity operator check
2. Check left hand side using top down inference.
3. If the lhs is not an optional, this is an *error* ⚠️.
4. Check right hand side using top down inference.
5. Find the common type of lhs and rhs. If there is no common type, this is an *error* ⚠️.
6. Try implicitly casting lhs and rhs to the common type, failuer is an *error* ⚠️
7. The type of the expression is the common type.

##### Ternary operator check

The `??` operator may not have `?:` or `?` on either side:

```c3
// Valid
(a ? b : c) ?? d;
a ?? (d ?: e);
// Invalid
a ? b : c ?? d
a ?? b ? c : d
a ?? b ?: c
```

##### Implicit binary casting

1. If the target type is a vector and the value is a scalar, and the value is implicitly castable to the vector element type, expand the value into a vector.
2. Proceed with regular implicit casting.

##### Signed implicit binary casting

This method of implicit casting takes a pair of signed and unsigned integers or integer vectors types of the same bitwidth and length.

1. If the value is an inline distinct type, flatten to its inline type.
2. If the value is not an integer / integer vector, this is an *error* ⚠️.
3. If the value is a signed integer / vector, attempt *implicit binary casting* to the signed type, 
4. If the value is an unsigned integer / vector, attempt *implicit binary casting* to the unsigned type.
5. If the cast fails, this an *error* ⚠️.

### Typed operator overload resolution

Three types of typed overload matches exist:

1. Exact match: the match is the expected type without conversion, this is the strongest match.
2. Conversion match: the match is possible after implicit conversion, this is the next strongest match.
3. Wildcard match: the match is on an untyped parameter, this is the weakest match.

Matches are preferred in order of strength, so exact -> conversion -> wildcard. So for example, an exact match will be preferred over any conversion or wildcard match.

Overloads are searched through all visible methods.

1. Find all possible matches.
2. Find the match or matches with the highest strength.
3. If there are more than one match with the highest strength, this is an *error* ⚠️.
4. If there is only one match, this is the resolved overload.

