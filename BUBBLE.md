
# Types
- `null` - `Option::None`
- `bool` - it's a boolean lol
- `int` - `i64`, upcasts to float when necessary
- `float` - `f64`
- `string` - `*const [u8]`, but owned and gc'd
- `list` - gc'd vec
- `map` - gc'd map of string hashes to values
  - maps can have an associated "fallback map" where things look if they fail to _get_ an index
    - setting an index on a map that doesn't have it will call the associated `std.operators.INIT` in the map if it exists
      - this will be called _even if you're just setting the index to `null`_
  - failed indexings return null
  - setting a map index to null drops it
  - does not store the original string, just the hash - be careful of collisions
- `function` - potentially a closure, can have upvalues
- `symbol` - a guaranteed-to-be-unique value - used for certain internal things, also hashable in maps
  - strings have the high bit clear and the 63 others as their hash, symbols have the high bit set and the 63 others as their internal ID

# Statements
- If: `if <expr> { <stmt>... } [else if <expr> { <stmt>... }]... [else { <stmt>... }]`
- Block: `<ident>: {}`
- Jump: `jump <ident>;` (Jumps to the start of a given block)
- Break: `break <ident>;` (Jumps past the end of a given block)
- Definition: `init <ident> = <expr>;` - variables are ALWAYS block scoped, except when their lifetime is extended by becoming an upvalue
- Expression: `[<ident> [<postfix>...] =] <expr>;`
- Return: `return [<expr>];`
- Try/catch: `try { <stmt>... } [catch <ident> { <stmt>... }] [finally { <stmt> }]`
	- Variables set in `try` will not be in scope in `catch` or `finally`
	- Variables set in `catch` will not be in scope in `finally`
	- Must have at least one of `catch` or `finally` 
- Throw: `throw <expr>;`
	- Anything _can_ be an error, but they're usually strings
	- Thrown errors walk the call stack collecting a trace, saving call sites, but if caught will drop them at the end of the `catch` block
	- Throwing an error inside a `catch` will prepend the caught error's trace to the current one
	- Uncaught errors will print out the string representation of the error (or `<string call failed>` if _that_ errors) followed by a traceback

## Syntax sugar
- For: `for (<ident> in <expr>) { <stmt>... }`
  - `init <ident> = <expr>; __for_UUID: { if <ident> != null { break __for_UUID; } <stmt>... <ident> = next(<ident>); jump __for_UUID; }`
  - behavior of `next()` is decided with a map's `std.operators.NEXT` key if it exists, defaulting to a pairwise iteration implementation for maps with none found
  - see `std.utils.Range` for going over a range of numbers
- While: `while [<cond>] { <stmt>... }`
  - `__while_UUID: { if <cond> { break __while_UUID; } <stmt>... jump __while_UUID; }`

# Comments
- Line: `# <line comment>`
- Block: `#[ <block comment> ]#`
## Doc comments (unenforced)
- Block: `#[[ <documentation> ]]#`

# Expressions
Pratt parse!
- Literals (`1`, `5.3`, `null`, `variableName`, `"meow"`, `$Symbol`) (`$Symbol != $Symbol`, the labels are for debugging and can be elided entirely with just `$`)
- Binary operations (`<expr> <binop> <expr>`)
  - `+ - * / % == != < > <= >= <=> & | ^` - all have `std.operators.` followed by their operator names
    - `<=>` usually returns `std.compare.LESS`, `std.compare.GREATER`, `std.compare.EQUAL`, or `null` depending on the comparison (all except null are symbols)
  - `&& ||` - not overloadable, only work on booleans
- Unary operations (`<pre> <expr> | <expr> <post>`)
	- Prefix:
	  - `!` - binary not
	- Postfix:
	  - `[<expr>, <expr>...]` - indexing (or calling the associated `std.operators.INDEX`)
	  - `.<ident>` - same as `["<ident>"]`
	  - `(<expr>, <expr>...)` - function calling (or calling the associated `std.operators.CALL`)
