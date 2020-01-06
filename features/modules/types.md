⬅ [🏠 Home](../../README.md)

⬅ [2 🛠 Features](../README.md)

⬅ [2.3 📦 Modules](README.md)

# 2.3.3 Type

A type describes the properties a value of that type should have.
Each such property is a definition.
A definition is then either a method or a immutable variable.
Only classes can implement types.

Say I have a server, I could define the type as follows:
```mamba
    type Server
        def connect:            (mut Self, IPAddress) -> Boolean throws [ServerErr]
        def last_sent_message:  (Self) -> String
        def send_message:       (mut Self, String) -> Boolean    throws [ServerErr]
        def disconnect:         (mut Self) -> Boolean
```

Now any class that implements `Server` must have these definitions.
```mamba
    class MyServer(def ip_address: IPAddress) isa Server
        def connected        <- false
        def mut last_message <- undefined

        def last_sent_message(self): String => self.last_message

        def connect (mut self, ip_address: IPAddress) -> Boolean throws [ServerErr] =>
            # perform some operations here
            self.connected <- true
            true

        def send_message(mut self, message: String) -> Boolean throws [ServerErr] =>
            # perform some operations here
            self.last_message <- message
            true

        def disconnect(mut self) -> Boolean =>
            # perform some operations here
            self.connected <- false
            true
```