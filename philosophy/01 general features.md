# General Features

Here I outline in short some of the core features of Mamba, and their motivation, and to reveal a bit of the philosophy behind the language.
These are covered in greater detail in their respective chapters. 

There are multiple programming paradigms in computer science.
A few of the popular one's are outlined in this short, non-exhaustive list:

Paradigm                     | Description
-----------------------------|-------------
Procedural Programming       | Use a set of functions, or procedures, to carry out computations.
Object Oriented Programming  | Model everything as an object. They may carry data, but on a conceptual level, they define behaviour.
Functional Programming       | Treat computation as a mathematical function, with no side-effects.

Each come with their own (or overlapping) set of philosophies and schools of thought. 
Certain languages stick to a single paradigm. 
SmallTalk, for instance, sticks to object oriented programming, and Haskell to functional programming.
Java has stuck mostly to object oriented, though it does make an exception for primitives, and has retroactively added features found in functional languages such as anonymous functions.

Mamba aims to provide a mix of the object oriented and functional programming paradigms.
It aims to improve upon Python by introducing static type checking, null safety, and a bunch of other features, such as type refinement.
We also use features from other languages which have proven successful over time.

## Syntax, Syntax Sugar, and Readability

One thing that can significantly hinder the readability of source code is syntax noise. 
Therefore, I tried to limit the amount of symbols and keywords in the language.
I believe that when reading a piece of code, one should be able to understand what is happening without being hindered by excessive syntax.

One way to make code "look" more elegant, and ideally more readable, is through use of syntax sugar.
This allows developers to, ideally, express their ideas in a more elegant manner. 
Syntax sugar however comes with its own set of problems, one of which is that it can introduce a new way of solving the same problem.
We therefore aim to use syntax sugar sparingly, but still often enough so the language does not become overly verbose.
A few examples are shown below:

- Functions and methods with no arguments, or a single argument, can be called without brackets.
- In a `foreach`, if we iterate over a collection of tuples, we can omit the brackets (just as in Python).
- We can define a range using `..` and `..=`.
- The negation of equality `=` is `/=`, instead of having to negate the entire equality using `not`.

As stated before, we also decrease the reliance on symbols, which may be another source of syntax noise.
This should make the language more closely map to the english language, but not so close as to introduce ambiguity into the language.
A few examples are given below:

- Use `and` instead of `&&`, i.e. `alice.is_online and bob.is_online` instead of `alice.is_online && bob.is_online`.
  We can even write `alice is_online and bob is_online`.
- Use `or` instead of `||`, i.e.  `foo() or bar()` instead of `foo() || bar()`.
- Use `not` instead of `!`, which can easily be missed when read. 
  So we write `if not productive then drink_coffee()` instead of `if !productive then drink_coffee()`.
- Use indentation to denote code blocks instead of `{` and `}`, just as in Python.

Mamba also uses arrow notation to more clearly denote the flow of data:
- `<-` is used to assign to variables. 
  It denotes data flowing from the expression on the right to the identifier on the left.
- `=>` is used to denote the control flow of the application.
  This is used for two reasons:
  - It more clearly differentiates data manipulation from application control flow.
  - It avoid ambiguity in the grammar by differentiating anonymous functions from control flow in certain situations.
- `->` is used within type definitions of functions and methods.

This, combined with (sparing) use of syntax sugar, should ideally make the language easier to read.
Take for instance the following piece of code:

    foreach composer in composers do
        if composer.death is undefined then
            println "[composer] has not died."
        else 
            def years_ago <- today - composer.death
            print "[composer] died [years_ago] years ago."
            
Without any knowledge of Mamba, the reader should ideally, when reading above, be able to deduce that:

- We are iterating over a set (or collection) of composers.
- We are checking whether a composer has died.
- If a composer has died, we print how long ago that was. 
  Presumably today is a date.

Notice how little program specific syntax there is:
 
- We use `[` and `]` to insert variables into strings.
- We use `:` to denote what type `ago` is.
- `if` and `else` are used for program flow, and `print` to print something to the screen.
  - Note how we use the postfix notation when calling `print`, so we don't need to wrap the whole string in brackets.
- Indentation is used to denote code blocks, making it easy for the eyes to follow what is being done where and when.

## Null Safety

The `null` value, or the `undefined` value in cases of Mamba, is value which is meant to symbolize the concept of nothing.
It can be useful in some cases.
Implemented poorly in a language however can make the language unwieldy to use, and can result in many bugs.
It has lead to headlines akin to:

> Null Pointers, the billion dollar mistake

Null values can either break the application flow of an application by throwing an exception, such as in Java, or simply result in undefined behaviour, such as in C++.
A language with no null safety basically nullifies much of the functionality of a type system.
If I write a method, and I say that it returns an integer, a user of said method should not have to worry about the return value potentially being `null`.
Over the years, multiple languages have tried to implement null safety, with the two prevailing strategies being as follows:

- The language does not contain `null` as a concept, but has a monad.
  This often called an `Optional`, and can either be `None`, or `Some`, with `Some` wrapping the desired value.
- Null safety is baked into the language, and is enforced by the type system itself.
  This is for instance the approach that Kotlin uses, and in my opinion the less verbose option as well.

Mamba bakes null safety into the language
`?` is appended to types that may be `undefined`. 
If this has not been appended, say in the signature of a method, this method may not return `undefined`.
`?.` can be used on expressions that may be null, and it means a method is only called if the expression is not `undefined`.

```
# g never returns undefined. 
# If we would try this we would get a type error.
def g(x: Int): Int -> if x > 2 then x + 2 else x - 2

# f may return undefined
def f(x: Int): Int? -> if x > 2 then x * 7 else undefined

def x <- f(2) ?. is_even() # is_even is called, and we get true
def y <- f(3) ?. is_even() # f(3) is undefined, so is_even() is never called and y is now also undefined

# note that both x and y have type Bool?, meaning that they may both be undefined.
```

As with most language, Mamba also uses bounds checking.
However, unlike other scripting languages such as Ruby or Python, I opted to raise an error when we attempt to access outside the bounds of an array (or collection), much like languages such as Java.
This should ideally make it easier to track down bugs, which might otherwise be undetected for some time.
This adheres somewhat to the **fail fast** philosophy.

## Error Handling

Handling errors is something that is difficult to do well in language.
As a developer, it can be difficult to determine where and when we should handle such errors.
Doing this elegantly, whilst at the same time remaining explicit, is also difficult.

We want to only have checked errors in our application. 
Once we run an application, we expect its behaviour to reflect what was written.

We also want to have a visual trace of what can go wrong within the source code. 
This makes it easier to identify potential causes of errors, as opposed to wrapping everything in for instance a `try` `catch`, which may bury errors or make it unclear what statement, or line, was the actual source of the error.
Therefore, we:
- Explicitly state where errors occur using `raises`, such as inline or in the function signature
- Use inline pattern matching, using `handle`, to encourage on-site error handling.

We outline how this would be used:

```
# We show that this function raises a negative number
def my_function(x: Int): Int raises [NegativeNumber] => if x < 0 then NegativeNumber else x * 10

def z <- 4 + 6
def y <- my_function(z) raises [NegativeNumber] # We state inline what this statement might throw

def z <- my_function(z) handle
    err: NegativeNumber => 
        print err # We could also log this error using a logger
        undefined
    num: other => num
```

Note that my_function could also return `undefined` directly.

## Mutability and Statefulness, and Immutability and Statelessness

Immutability, which allows us to change a value of a variable, brings with it great flexibility.
In certain situations however, this flexibility may not be desirable and can be abused.
Therefore, we want to be able to define something as immutable, meaning that it cannot be changed.

When a variable is immutable however, it should truly be immutable. 
Often I see that even when a variable is declared immutable, it is still possible to modify it's internal fields.

Therefore, must like `Rust`, we specify whether `self` is mutable in the signature of a method.
This means that methods which modify the internals of an object, can only be called if said object is mutable.

```
stateful MyStateful
  def mut field <- 10

  # can only be called if self is mutable
  def method_1(mut self) => self.field <- 20 # we can assign because self is mutable

  def method_2(self) => print "This doesn't modify anything."
```

`method_1` specifies that self must be mutable for it to be called.
`method_2` may not modify the fields of `self` as `self` is not mutable.

We also make a distinction between State and Statelessness, using `stateful` and `stateless`.
Statlessness is often seen in functional programming languages.
It also allows us to enfore the following rule: `forall f, x = y -> f(x) = f(y)`, as is the case in functional programming language.
This might seem obivous to mathmaticians, but is not always the case when state is rought into play.
Take the following

```
stateful MyStateful 
  def mut called_before <- false

  def f(x: Int): Int => if called_before then x * 2 else x * 3
```

Now we call the same method twice, and we see that it returns two different values:
```
def my_stateful <- MyStateful()

my_stateful.f(10) # retuns 20
my_stateful.f(10) # return 30
```

Ensuring that something has no state allows us to enforce the above rule, as the same function or method will always return the same value for the same input.
In a `stateless`, we cannot have mutable top level definitions, which allows use to enforce this rules as its internal state can never change.

```
stateless MyStateless
  def f(x: Int): Int => x * 2
```  

Now we always get the same value:
```
def my_stateless <- MyStateless()

my_stateless.f(10) # returns 20
my_stateless.f(10) # still returns 20
```

## Static Typing and Type Inference

Often a distinction is made between static and dynamic typing. 

If the application were dynamically typed, we would constantly have to verify that variables are indeed what they claim to be.
Say I have a variable `beethoven`, can I assume that it is an instance of `Composer`?
   
    # how can I be sure that this function argument is a composer?
    def my_function (composer) => composer.composer_method()
    
There are of course ways to check this, such as using the `isinstance` method in Python, but this is rather tiresome.
To this end, we use types. 
A user defines a class `Composer`, which defines the behaviour of a composer:

    stateless Composer
        def composer_method(): Int
    
And then we define the `my_function` as such:

    def my_function (composer: Composer): Int => composer.composer_method()
    
Now, in the body of the function, we can rest easy knowing that the passed variable is indeed a composer. 
It is actually now impossible to pass another variable type to the function, as this is statically checked by the type checker.
If it sees that we try to pass something that is not a composer, will give an error, meaning that the program wil not run.

In some programming languages, we have to explicitly state the type of each variable. 
This however makes the application rather verbose. 
Take for instance:

    def x: Int <- 10                     # x is obviously an integer
    def c: Complex <- Complex(10, 20)    # from the right hand side it is already clear that c is complex

Instead, we can use type inference. 
The type of every variable is inferred from the context in which it is used.

```
    def x <- 10                 # x has type Int, we know this because 10 is an Int
    def c <- Complex(10, 20)    # c has type Complex
    def y <- 20.1               # 20.1 uses decimal notation, so we know y is a real number, or Real
    
    def z: Real <- 10.5         # In some situations however, you still might want to explicitly mention the type
```

The program is still statically typed, but now we don't require the developer to write everything out in full.

### Type Aliases and Type Refinement

We can also use type aliases and type refinement to further refine types by adding conditions to them.
Say we have the following:

    type DeadComposer <- Composer where
        self.death /= undefined else Err("Composer is not dead.")

We now rewrite my_function so it only works for `DeadComposer`s:

    def my_function (composer: DeadComposer): Int => today.year - composer.death.year
    
Again, we can rest assured that `composer` is a `DeadComposer` in the body of the function. 
To use such a function, we must explicitly cast a `Composer`:

```
    def chopin <- Composers("Chopin")
    
    if chopin isa DeadComposer then
        def years_ago <- my_function(chopin)                    # chopin is dynamically casted to a DeadComposer
        print "[chopin.name] died [years_ago] years ago."
```

This draws on concepts of **Design by Contract** philosophy.

Furthermore, it also allows us to explicitly define the state of an object, something which is often left ambiguous.
For instance, we can say a server is connected or disconnected by doing the following:

```
    type Server
        def private connected: Boolean
        def send_message(self: ConnectedServer, message: String): String

    type ConnectedServer isa Server where
        connected else Err("Server is not connected")
```

And we may then elsewhere implement this `Server` interface:

    stateful MyServer isa Server
        def private connected <- false
        
        def init() -> ...
        
        def connect(ip: IpAddress) -> ...
        
        # You can only call this function if I am a connected server
        def send_message(self: ConnectedServer, message: String) -> ...

This is a rather trivial example, but it shows how we can explicitly name the different states of a server.
