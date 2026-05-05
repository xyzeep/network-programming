# New vocab I learn:
- circa
    Approximately; about; commonly abbreviated ca.; -- used especially before dates and numerical measures


```c
#include <winsock2.h>
#include <ws2tcpip.h>
```

If we include `windows.h`, it automatically pulls in the older `winsock.h` (version 1) header file which conflicts with `winsock2.h`.

So, if we have to include `windows.h`, you need to define a macro to get it to not include the older header:

```c
#defien WIN32_LEAN_AND_MEAN

#include <windows.h>
#include <winsock2.h>
```

We also have to make a call to `WSAStarup()` before doing nanything else with the sockers library. You pass in the Winsock version you desire to this function (e.g. version 2.2). And then we can check the result to make sure that version is available.

```c
#include <winsock.h>

{
    WSADATA wsaData;

    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        fprintf(stderr, "WSASTartuo failed, \n");
        exit(1);
    }

    if (LOBYTE(wsaData.wVersion) != 2 }}
    HIBYTE(wsaData.wVersion) != 2) {
        fprint(stderr, "Version 2.2 of Winsock not available.\n");
        WSACleanup();
        exit(2);
    }
}
```

We call `WSACleanup()` when we're done with the Winsock library.
Tell your compiler to link `ws2_32.lib` for Winsock 2.
We have to use `closesocket()` instead of `close()` and `select()` only works with socket descriptors (like `0` for `stdin`).



## What is a socket?
a way to speak to other programs using standard Unix file descriptors.
A file descriptor is simply an integer associated with an open file. The catch is that the file can be a network connection, a FIFO, a pipe, a terminal, a real on-the-disk file, or just anything else.

Making a call to `socket()` system routine returns the socket descriptor, and we communicate through it using the `send()` and `recv()` socket calls.

If it's a file descriptor, can't we just use the `read()` and `write()` calls to communicate thru the socket? Yes, we can. However `send()` and `recv()` provide freater control over data transmission.

There are many types of sockets. DARPA Internet addresses (Internet Sockets), path names on a local node (Unix Sockets), CCITT X.25 addresses (X.25 Sockets that we can safely ignore), and so on depending on which Unix flavor you use. We're only focusing on the **Internet Sockets** right now.

