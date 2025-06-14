# Objection

Objection is a (currently unimplemented) interpreted programming language with one tenet: anything and everything is an object.

Objective C's syntax highlighting will be used for examples here, as it fits pretty well.

## Types
### Primitives
- `char`, `uchar`, `short`, `ushort`, `int`, `uint`, `long`, `ulong` \
    Stores an integer with varying precision. \
    Example: `13uc`, `-16s`, `717`, `8305ul`
- `float`, `double` \
    Stores a floating point number with varying precision. \
    Example: `13.5f`, `1e20d`
- `string` \
    Stores a string of text. \
    Example: `"Hello there!"` \
    All strings have `.format(any... value)` to allow string formatting. \
    Also, newlines can be inserted into strings with no difficulty.
- `bool` \
    Stores a 1-bit value. \
    Example: `true`, `false`
- `vec` \
    Stores an ordered, immutable group of values. \
    Example: `(1, 2, 3)`
- `list` \
    Stores an ordered, mutable group of values. \
    Example: `[1, 2, 3]`
- `group` \
    Stores an unordered, immutable group of values. \
    Example: `|(1, 2, 3)|`
- `set` \
    Stores an unordered, mutable group of values. \
    Example: `|[1, 2, 3]|`
- `any` \
    Abstract, can represent any type of value. \
    Variables of this type aren't type-checked.
### Functional
- `annotation` \
    The annotation for a type or function. \
    Example: `<int>`, `<float, <int, bool>>`
- `argument` \
    An argument specifier in a function signature. \
    Example: `x: int`, `y: function<int>`
- `type` \
    A class. Will touch on later. \
    Examples: `{ public x: int = 6; }`, `int`, `bool`
- `signature` \
    The concatenation of a `vec<annotation>` and an `argument`. \
    Example: `(x: int)<float>`
- `function`
    The concatenation of an `argument` and a `type`. \
    Example: `<int>{return 5;}`
- `callable`
    The concatenation of either a `vec<annotation>` and a `function`, or a `signature` and a `type`. \
    Example: `(x: int)<int>{return x * 2;}`
- `partial`
    The concatenation of a `callable` and a `vec<any>`. Not ran yet. \
    Example: `(x: int)<int>{return x * 2;}(5)`
- `runner`
    Only the built-in value `!` can have this type. Runs a `partial` when concatenated with one. \
    Example: `!((x: int)<int>{return x * 2;}(5)) == 10`
### Syntax
_Notice how this is still in the Types section._
- `keyword`
    A keyword. New ones can't be made (how would that work?) \
    Examples: `if`, `for`
- `operator`
    A two-argument operator. \
    All operators implement `concat = (other: std::ops::assign)<operator>` to allow in-place mutation. \
    Example:
    ``` objective-c
    (5 (operator)->{ // subclass of operator, will be touched on later
      public operate: callable<int> = (a: int, b: int)<int>{return a+b;};
    } 8) == 13
    ```

## Built-ins
- `null` \
    `any` - Represents the abscence of a value.
- `!` \
    `runner` - Runs a `partial` when concatenated with one.
- `public` \
    `keyword` - Allows a value of a type to be accessed externally.
- `+`, `-`, `*`, `/`, `%`, `**` \
    `operator` - Arithmetic operators.
- `==`, `~=`, `<`, `>`, `<=`, `>=` \
    `operator` - Comparison operators.
- `&&`, `||`, `^`, `~` \
    `operator` - Bitwise operators. `~` represents not.
- `import` \
    `keyword` - Allows importing a file as a type. \
    Example: `import (utils: "./utils.ob");`
- `print` \
    `callable` - Outputs to stdout. \
    Example: `!print("Hello, world!\n");`
- `input` \
    `callable` - Gets a line of input from stdin. \
    Example: `!input("Input your name: ");`


_ TODO: Finish _
