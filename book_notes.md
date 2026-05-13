# New vocab I learn:
- circa: Approximately; about; commonly abbreviated ca.; -- used especially before dates and numerical measures
- naysayer: 1. One who consistently denies, criticizes, or doubts; a detractor. 2. Someone with an aggressively negative attitude.
- automagically



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



# What is a socket?
a way to speak to other programs using standard Unix file descriptors.
A file descriptor is simply an integer associated with an open file. The catch is that the file can be a network connection, a FIFO, a pipe, a terminal, a real on-the-disk file, or just anything else.

Making a call to `socket()` system routine returns the socket descriptor, and we communicate through it using the `send()` and `recv()` socket calls.

If it's a file descriptor, can't we just use the `read()` and `write()` calls to communicate thru the socket? Yes, we can. However `send()` and `recv()` provide freater control over data transmission.

There are many types of sockets. DARPA Internet addresses (Internet Sockets), path names on a local node (Unix Sockets), CCITT X.25 addresses (X.25 Sockets that we can safely ignore), and so on depending on which Unix flavor you use. We're only focusing on the **Internet Sockets** right now.

## Internet Sockets
We're gonna look at two types of internet sockets. Stream Sockets and Datagram Sockets, which are referred to as "SOCK_STREAM" and "SOCK_DGRAM", respectively. Datagram sockets are sometimes called connectionless sockets. (Though they can be `connect()`'d if you reallt want.)

Stream sockets are reliable two-way connected coomunication streams. If you output two items in the sockets in the order "1, 2", they will arrive in the same same order at the opposite end. They will also be error-free. `telnet` and `ssh` applications use stream sockets. Web browsers use the HTTP which uses stream sockets to get pages.

Stream sockets achieve this high level of data transmission quality because they use a protocol called "The Transmission Protocol", otherwise known as TCP. TCP makes sure your data arrives sequentially and error-free.

About UDP, if you send a datagram, it may arrive. It may arrive out of order. If it arrives, the data within the packet will be error-free. Datagram Sockets also use IP for routing, but they don't use TCP; they use the "User Datagram Protocol", or UDP.

They are connectionless because we don't have to maintain an open connection as we do with stream sockts. We just build a packet, slap ad IP header on it with destination information, and send it out. No connection needed. However, UDP is unreliable by itself. Packets can be lost or arrive out of order. Some appliations like TFTP and DHCP still use UDP for speed but they add their own protocol system on top of UDP. For example, the tftp protocol says that each packet that gets sent, the recipient has to send back a packet that says, "I got it!" (an "ACK" packet). If the sender gets no reply in, say, five seconds, he'll re-transmit the packet until he finally gets an ACK. This ACK process is very important when implementing reliable `SOCK_DGRAM` applications. Unreliable applications like games, audio, or video, we just ugnore the dropped packets, or pehaps try to compensate for them.

Why would we use an unreliable underlying protocol? Speed, speed, and speed. It's way faster to fire-and-forget then it is to keep track of wht has arrived safely and make sure it's in order and all that. If you're sending chat messages, TCP is great; if you're sending 40 positional updates per second of the players in the worlld, maybe it doesn't matter so much if one or two get dropped, and UDP is a great choice.


## Low level nonsense and Network Theory
A packet is born, the packet is wrapped ("encapsulated") in a header (and rarely a footer) by the first protocol (say, the TFTP protocol), then the whole thing (TFTP header inclded) is the next (IP), then again the final protocol on the hardware (physical) layer (say, ethernet). When another computer receives the packet, the hardware strips the Ether header, kernel strips the IP and UDP headers, the TFTP program strips the TFTp header, and it finally has the data.

### ISO/OSI: Layered Network Model

- Application
- Presenation
- Session
- Tranport
- Network
- Data Link
- Physical

The physical layers is the hardware (serial, Ethernet). Application layer is where users interact with the network.
 
 A layered model more consistent with Unix might be:
- Application Layer (telnet, ftp)
- Host-to-Host transport Layer (TCP, UDP)
- Internet Layer (IP and routing)
- Netwrok Acess Layer (Ethernet, wifi)

All we have to do for stream sockers is `send()` the data out. All we have to do for sockets is encapsulate the packet in the method of your choosing and `sendto()` it out. The kernel build the Transport Layer and Internet Layer on for you and the hardware does the Network Access Layer.


# IP Addresses, `struct` s, and Data Munging

IPv4: 4 bytes (32 bits). Around 4 billions addresses.
Vint Cerf (Father of the Internet) warned us that we're going to run out of IPv4 addresses. Represented in numbers and dots like so: `192.0.2.111`
IPv6: 16 bytes (128 bits). Around 340 trillion trillion trillion addresses. Represented in hex and colons like so: `2001:0db8:c9d2:aee5:73e3:934a:a5ae:9551`

Lots of times, there are so many zeros in an IPv^ address, so we write it in a compressed form. Example:
```
2001:0db8:c9d2:0012:0000:0000:0000:0051
2001:db8:c9d2:12::51

2001:0db8:ab00:0000:0000:0000:0000:0000
2001:db8:ab00::

0000:0000:0000:0000:0000:0000:0000:0001
::1
```

`::1` is the loopback address. In IPv4, loopback address is `127.0.0.1`


We can also represent an IPv4 address in IPv6 format using the following notation: "`::ffff:192.0.2.33`"

### Subnets
An IP address has two parts: network portion and host portion. For example, in `192.0.2.12`, if the first 3 bytes are the network, the last byte (12) is the host. The network address is `192.0.2.0` (host byte zeroed out).

Old system used classful networking

Class A: 1 byte network + 3 bytes host = ~16 million hosts
Class B: 2 byte network + 2 bytes host 
Class C: 3 byte network + 1 bytes host = ~ 256 hosts

#### Netmask:
Defines which part of the IP is the network. Bitwise AND the IP with the netmask to ge the network address.
Example: `192.0.2.12 AND 255.255.255.0 = 192.0.2.0`

CIDR Notation (new stlye): A way to represent IP address and its associated subnetmask. Netmask can be any number of bits. It is written as a suffix after the IP with a slash(`/`). Example: `192.0.2.12/30`. Here, `/03` means 30 network bits and remaining (2 bits) for possible hosts.
Works the same for IPv6 too. A netmask is always 1-bits first, then 0-bits, never mixed, i.e., some number of 1's followed by some number of 0s.

### Port numbers
besides IP address (used by the IP layer), there is another address (port number) that is sued by TCP (stream sockets) and UDP (datagram sockets). It's a 16-bit number that's like the local address for the connection.

## Byte Order
- Big-Endian: stores the big end first. `b34f` is stored as `b3 4f`. THis is Netwrok Byte Order (what the internet uses). Most significant bit in the smallest memory address.
- Little-Endian: stores bytes reversed. `b34f` is stored as `4f b3`. Intel/Intel-compatible processores do this. Least significant bit on the smallest memory address.

**Host Byte Order** is what you machine uses natively
**Network Byte Order** is always Big-Endian

We always convert to Network Byte Order before sending data, and convert back to Host Byte ORder upon receiving.

Conversion functions: These handle short (2 bytes) and long (4 bytes):

htons()             ->      host to network
shorthtonl()        ->      host to network
longntohs()         ->      network to host
shortntohl()        ->      network to host long



### Private (or disconnected) Networks

Many places use a firewalls are used to hide the network from the rest of the world for thier own protection. Often times, the firewall translates "internal" IP addresses to "external" using a process called **Network Address Translation**, or NAT.



# System Calls

## `getaddrinfo()`: Prepare to launch!
It prepares the network address information you need before making a connection. It doesn't connect or listen by itself.
You give it a hostname (like `"www.examplme.com"`) anda port/service (like `"80"` or `"http"`), and it returns a linked list of `struct addrinfo` results, each containing a ready-to-use `struct sockaddr`.

## `socket()`: Get the File Descriptor!
```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol)
```

`domain` is `PF_INET` or `PF_INET6`,
`type` is `SOCK_STREAM` or `SOCK_DGRAM`,
and `protocol` can be set to `0` to chhose the proper protocol for the given `type`. Or we can use `getprotobynamme()` to look up the protocol we want, "tcp" or "udp".


`socket()` simply returns a *socket* descriptor that we can use later in system calls, or a -1 on error.

## `bind()`: What port am I on?
Once we have the socker, we might have to associate that socket with a port on our local machine. This is commonly done if we're going to `listen()` for incoming connections on a specific port. The port number is used by the kernel to match the incoming packet to a certain process's socket descriptor. If we're only going to `connect()` (when we're the client, not the server), this is probably not necessary.

Synopsis:
```c
#include <sys/types.h>
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
```
`sockfd` is the file descriptor returned by `socket()`. `my_addr` is a pointer to a `struct sockaddr` that contains information abt your address, name port and IP adddress. `addrlen` is the length in bytes of that address.

## `connect()`: Hello!

```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
```

`sockfd` is our socket file descriptor returned by the `socket()` call, `serv_addr` is a `struct sockaddr` containing the destination port and IP address, and `addrlen` is the length in bytes of the server address structure.






















