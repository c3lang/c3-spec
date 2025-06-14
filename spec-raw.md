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


