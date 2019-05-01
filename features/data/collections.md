⬅ [🏠 Home](../../README.md)

⬅ [2 🛠 Features](../README.md)

⬅ [2.2 📝 Data](README.md)

# 2.2.1 Collections

The grammar has Four types of collections:
- `Set`
- `List`
- `Tuple`
- `Map`

A collection may be mutable. 
However, this does not mean that its items are mutable. 
To make the items mutable as well the `ofmut` keyword must be used.

### Set

A set is created using either `{` and `}`, or by using the set-builder notation.
A set is an unordered collection of unique items.

Below we show some examples of how a set may be used:
```
    # A set may be immutable
    def animals <- { "dog", "cat", "mouse" }
    # Or mutable
    def mut instruments <- { "piano", "violin", "flute" }    
    
    # You can also define an immutable set of mutable variables
    def countries ofmut <- { "Netherlands", "Belgium", "Luxembourg" }
    # Or you can even define a mutable set of mutable variables
    def mut composers ofmut <- { "Beethoven", "chopin", "mozart" }
  
    # I can iterate over a set
    for animal in animals print animal
    
    # I can add an item to a mutable set
    instruments add "trombone"
    # Or remove an item from a mutable set
    instruments remove "violin"
    
    # I can change the values of items in a set of mutable items
    countries get "Netherlands" add " - Amsterdam" 
    for country in countries print country
```

### List
A list is created using `[` ... `]`, or by using the list-builder notation.

A list can be accessed using an index: `list(1)`\
A list may be ordered.

### Tuple

A tuple is created using `(` ... `)`.

A tuple a collection of named items.

### Map

A map is created using `{` ... `}`, where each mapping is represented as such: `key -> value`, or a map-builder.

A map is a collection of unordered set of items.
