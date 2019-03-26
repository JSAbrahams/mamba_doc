⬅ [🏠 Home](../../README.md)

⬅ [2 🛠 Features](../README.md)

⬅ [2.3 📦 Modules](README.md)

# 2.3.3 State and Stateless

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
  def method_1(self: mut) => self.field <- 20 # we can assign because self is mutable

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

A class describes the properties of an instance of that class. A class encapsulates data and contains definitions. A 
definition is either a value, or an immutable method. A definition may be private.

An instance of a class may be created. A class may not inherit from another class, but may have another class as a 
property, and forward its methods. A class may implement a type. It must then either implement the methods described in
the type, or contain an instance of a class that forwards these methods.

## Using Type Aliases to Define State of Self

In certain situations, we want to make sure that certain methods can only be called when an instance of a class is in a
certain state. This can be achieved using type aliases and type refinement.

Say I have a type `Server`:

    type Server 
        def connected:          Boolean
        def ip_address:         IPAddress
        def mut last_message:   String
        
        def connect:            (IPAddress) -> Boolean throws [ConnectionErr]
        def last_sent_message:  () -> String
        def send_message:       (String) -> Boolean    throws [ConnectionErr]
        def disconnect:         () -> Boolean
        
    type ServerErr(msg: String) isa Err(msg)

We can do the following:

    type ConnectedHTTPServer isa HTTPServer when
        self connected else ServerErr("Not connected.")
        
    type DiconnectedHTTPServer isa HTTPServer when
        self not connected else ServerErr("Already connected.")
        
    stateless HTTPServer
        def stateless_message => "This message is always the same."
        
    stateful HTTPServer isa Server
        def connected <- false
        def mut last_message <- ""
        
        def init(self: DisconnectedHTTPServer, def ip_address: IPAddress)
        
        def last_sent_message(self): String => self last_message
        
        # self must be disconnected
        def connect (self: DisconnectedHTTPServer, ip_address: IPAddress): Boolean =>
            # perform some operations here
            connected <- true
            true
            
        # self must be mutable to send message
        def send_message(mut self: ConnectedHTTPServer, message: String): Boolean =>
            # perform some operations here
            self last_message <- message
            true
            
        # self must be connected to disconnect
        def disconnect(mut self: ConnectedHTTPServer): Boolean =>
            # perform some operations here
            self.connected <- false
            true
