I am reading [this book](https://beej.us/guide/bgnet/) to learn about network programming and implement an HTTP server from scratch. This repo will contain the notes, implementation, and any related extras.

---

## By the end, we must understand:
- How TCP connections work (handshake, data transfer, close)
- What HTTP/1.1 actually looks like as raw text
- How to parse an HTTP request by hand (method, path, headers, body)
- How to construct a valid HTTP response
- How socket(), bind(), listen(), accept() work in C
- How to handle multiple clients (at minimum, sequentially)
- How to manage memory carefully when dealing with buffers
- What a file descriptor is and how the OS uses them

## Steps
1. Echo server first. Write a server that accepts a TCP connection and sends back whatever it receives. No HTTP. Just get sockets working.
2. Read a raw HTTP request. Use curl or a browser to hit your server and print exactly what comes in. Study that text.
3. Parse the request line. Extract the method (GET, POST) and the path. Ignore everything else for now.
4. Send a hardcoded response. Return a valid HTTP/1.1 200 response with "Hello World" as the body. Get a browser to render it.
5. Add basic routing. If path is /, respond one way. If /about, respond another. Return 404 for anything else.
6. Parse headers. Read Content-Type, Content-Length, and anything else you need.
7. Handle a POST request with a body. Read the body using Content-Length. This is where buffer management gets real.
8. Serve a static file. Read a file from disk and send it as the response body. Set the right Content-Type.
9. Handle multiple clients. Start with a simple fork() per connection. Later, look into select() or poll() for non-blocking I/O.
10. Test it hard. Use curl, a browser, and write your own test scripts. Try to break it.

---

What is a socket?
-> that serves as an endpoint for sending and receiving data across a network.
It is combined of an IP addess, port number, and a transport protocol (TCP or UDP).

Core loop the server runs:
```
create socket
bind to address + port
listen for connections
loop forever:
    accept a connection
    read the request
    parse it
    build a response
    send the response
    close the connection
```

That's the skeleton of every HTTP server ever written.


