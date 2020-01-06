⬅ [🏠 Home](../../README.md)

⬅ [2 🛠 Features](../README.md)

⬅ [2.3 📦 Modules](README.md)

# 2.3.4 Type Aliases

In certain situations, we want to make sure that certain methods can only be called when an instance of a class is in a certain state.
This can be achieved using type aliases and type refinement.


I have a type `Server`:
```mamba
    Type Server
        def connect:            (mut Self, IPAddress) -> Boolean throws [ServerErr]
        def last_sent_message:  (Self) -> String
        def send_message:       (mut Self, String) -> Boolean    throws [ServerErr]
        def disconnect:         (mut Self) -> Boolean

    type ServerErr(msg: String) isa Err(msg)
```

And I define the following type aliases:
```mamba
    type ConnectedHTTPServer isa HTTPServer when
        self connected else ServerErr("Not connected.")

    type DiconnectedHTTPServer isa HTTPServer when
        self not connected else ServerErr("Already connected.")
```

We can do the following:
```mamba
    class HTTPServer(mut self: DisconnectedHTTPServer, def ip_address: IPAddress) isa Server
        def connected        <- false
        def mut last_message <- undefined

        def last_sent_message(self): String => self last_message

        def connect (mut self: DisconnectedHTTPServer, ip_address: IPAddress) -> Boolean throws [ServerErr] =>
            # perform some operations here
            self.connected <- true
            true

        def send_message(mut self: ConnectedHTTPServer, message: String) -> Boolean throws [ServerErr] =>
            # perform some operations here
            self.last_message <- message
            true

        def disconnect(mut self: ConnectedHTTPServer) -> Boolean =>
            # perform some operations here
            self.connected <- false
            true
```
