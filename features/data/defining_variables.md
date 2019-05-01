⬅ [🏠 Home](../../README.md)

⬅ [2 🛠 Features](../README.md)

⬅ [2.2 📝 Data](README.md)

# 2.2.2 Defining Variables and Functions (or Methods)

Every definition must be preceded with the `def` keyword. 
This is not necessary when reassigning however.

Functions and methods cannot be reassigned, mutable values can however. 
A value is an expression which may be evaluated.

## Variables

A variable `x` is assigned to as such:\
`def x <- <expression>` or `def mut x <- <expression>`

## Methods

// TODO write doc methods
    
### Postfix Notation

When a method or function has only one argument, we can use postfix notation when writing calling that function or 
method:

    my_function(10)
    my_function 10 # also works fine
    
    my_object.my_method(10)
    my_object.my_method 10  # this works too
    my_object my_method 10  # also works
    my_object my_method(10) # we can mix and match if we really want

### Default values

We can have default values:

    class MyClass
        def my_field <- 5
        
        def my_method(x: Int, y: Int <- 2) => self my_field <- x + y

We can now call the method as such:

```
    def my_class <- MyClass()
    
    my_class.my_method(10, 2) # works fine
    my_class.my_method(10)    # exactly the same arguments as the function call above
    my_class my_method 10     # using postfix notation
    my_class my_method(10)    # also works
```

### Default behaviour

We can assign default behaviour to a method or function. 
To demonstrate this, we will use a toy factorial example. 
You might first write it as such:

    def factorial(n: Int) =>
        if n = 0 then 1
        else n * factorial (n - 1) 

However, we could make this look much better with default behaviour. 

    def factorial (n: Int) => n * factorial (n - 1) # for all other values of n, this function is called
    def factorial (0)      => 1                     # if n is 0, then this function is called instead

As long as a version exists of a function or method with arguments this is allowed.
