## Types

- Null (`null`) - Empty singleton
- Boolean (`boolean`) - bool
- Byte (`byte`) - u8
- Integer (`integer`) - i64
- Float (`float`) - f64
- String (`string`) - Resembles an `Arc<[u8]>`, but managed by a GC
- List (`list`) - Resembles an `Arc<Mutex<Vec>>`, but managed by a GC
- Map (`map`) - Resembles an `Arc<Mutex<HashMap>>`, but managed by a GC
- Object (`object`) - Primitive for complex behavior, use things like `object=>set_fallback(new object { }, <type>)`, or for shorthand `object=>of(<type>)`
- Type (`type`) - Primitive for defining behavior of `object`s
- Function (`function`) - Type factory that generates types used for everything defined using `fn`
- Symbol (`symbol`) - Unique singletons

## Syntax quirks

- Can do `new <ty> [as <ident>] { ~? contents ?~ }` for `<ty>` being any `type`, and any `<ident>`, to make an instance
  - Any variables initialized within the block are set as fields when the block ends
- Can do `fn as <ident> (arguments) -> ty { ... }` to allow easier recursion
- Comments are `??`, block comments are `~? ... ?~`
- Use `=>` to access fields of a variable's type, and `.` to access fields of a variable

## Example behavior

```
Person := new type as This {
    MAX_AGE := 120;
    DEFAULT_NAME := "John Doe";
    init := fn(name: string, age: int) -> object: {
        obj := object { name: name ? "John Doe", age: age ? 20 };
        object=>set_fallback(obj, This);
        object=>freeze_fallback(obj);
        return obj;
    };
    check_age := fn(self: This) -> bool: {
        return self.age > This=>MAX_AGE;
    };
    ?? static variables set to types signifies that a field of that type is expected on instances, but this is not enforced - accessing a field that doesn't exist gives null
    ?? statics set to types with SCREAMING_CASE are associated types
    age := string;
    name := int;
    sample := function=>taking([string, int])=>returning(object); ?? <- this is a type
}

?? set_fallback makes any field accesses "fall back" to the specified type if none exists
?? the fields that things like set_add on Operable set behind the scenes are unnameable
?? It sets, for example, symbol=>set(Person, $ADD_OPERAND, <the_function>), but it's not actually called $ADD_OPERAND
object=>set_fallback(Person, Standard=>Types=>Operable);

?? Very silly implementation, but gets the point across
Person.set_add(fn(self: Person, other: Person) -> Person: {
    return Person=>init(self.name.concat(other.name), self.age + other.age);
});

?? Even past this, the unnameable field set_add set in the type remains
object=>clear_fallback(Person);
object=>freeze_fallback(Person);
object=>hide_fallback(Person); ?? now, calling object=>get_fallback(Person) would throw an instance of Standard=>Exceptions=>HiddenFallbackException
```

## Weak symbol keys?
```
Sample := null;
{
    key := new symbol {};
    Sample = new type {};
    symbol=>set(Sample, key, 5);
    Sample.$key = 5; ?? Maybe equivalent shorthand syntax?
    ?? If this is the case, maybe allow $key = 5 as well - but both of these only when key is a symbol, raise an exception otherwise
}
?? Sample.$key can now be GC'ed, as there is no way to access it - its key was dropped, and fields hold a weak reference to their identifiers

s := symbol {};
$s = s;
drop s; ?? $s needs to be GC'ed here
```
