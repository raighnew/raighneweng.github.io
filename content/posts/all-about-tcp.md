+++
title = 'All About TCP and TLS Connection'
date = 2024-03-27T21:19:23+08:00
draft = false
+++

This article will be continually updated.

## TCP handshake

TCP need three-way handshake to establish connection and four-way handshake to termination. Here's what we learned from the book, to establish the connection it needs 1.5 RTT (three-way handshake) and to terminate it needs 2 RTT (four-way handshake). The progress details looks like this:

Connection Establishment:
```bash
Client(SYN-Sent)     >     SYN        >    Server
Client(Establish)    <   SYN + ACK    <    Server(SYN-Received)
Client(Establish)    >      ACK       >    Server(Establish)
```

Connection Termination:
```bash
Client(FIN-Wait-1)   >     FIN       >    Server
Client(FIN-Wait-2)   <     ACK       <    Server(CLOSE-WAIT)
Client(FIN-Wait-2)   <     FIN       <    Server(LAST-ACK)
Client(TIME-Wait)    >     ACK       >    Server(CLOSE)
Client(Wait For DoubleMaximum Segment Life (MSL) Time)
```

### Modern TCP Handshake
The aforementioned process is based on traditional methods from textbooks dating back 30 years. With evolving technological needs, there have been some practical changes when the [TCP delayed acknowledgment](https://en.wikipedia.org/wiki/TCP_delayed_acknowledgment) feature is enabled on the server side. Connection establishment still requires 1.5 RTTs, but the final ACK can contain the initial request data. Termination now only requires around 1.5 RTTs.

Connection Establishment:
```bash
Client(SYN-Sent)     >     SYN        >    Server
Client(Establish)    <   SYN + ACK    <    Server(SYN-Received)
Client(Establish)    >   ACK + Data   >    Server(Establish)
```

Connection Termination:
```bash
Client(FIN-Wait-1)   >     FIN       >    Server
Client(FIN-Wait-2)   <   FIN + ACK   <    Server(LAST-ACK)
Client(TIME-Wait)    >     ACK       >    Server(CLOSE)
Client(Wait For DoubleMaximum Segment Life (MSL) Time)
```

# TLS handshake
HTTP relies on TCP, IP, and Link Layer, HTTPS relies on HTTP and TLS, so TLS also need handshake based on TCP three-way handshake. And it behaviors different on different TLS version.

## TLS 1.2 requires 2 RTTs for connection establishment

Connection Establishment:
```bash
...(TCP three-way handshake)
Client >  Say Hello                               >  Server
Client <  Say Hello, and ask cert                 <   Server
Client >  Client key exchange, choose cipher Spec >  Server
Client <  Change cipher Spec, finish              <   Server
```

## TLSv1.3 needs 1 RTT for connection establishment

Connection Establishment:
```bash
...(TCP three-way handshake)
Client >  Say Hello, Key share                                 >  Server
Client <  Say Hello, Key share, and verify certificate, finish <  Server
```

## Accelerating the Handshake: Is Simultaneous TLS and TCP Handshake Possible?

Indeed, handshakes can be expedited under the following conditions:

1. Both the client and server have TCP Fast Open and TLS 1.3 enabled.
2. There has been at least one prior connection between the client and server (to establish a TFO Cookie).


### TFO Cookie
When TCP Fast Open is enabled on both the client and server, it can shave off 1 RTT in subsequent requests.

Initial Connection with TFO Cookie:
```bash
Client(SYN-Sent)     >     SYN + with Cookie request    >    Server
Client(Establish)    <     SYN + ACK + with Cookie      <    Server(SYN-Received)
Client(Establish)    >         ACK + Data               >    Server(Establish)
```

Subsequent Connection with TFO Cookie:
```bash
Client(SYN-Sent)     >     SYN + with Cookie + Data    >    Server
Client(Establish)    <     SYN + ACK + verify Cookie   <    Server(SYN-Received)
Client(Establish)    <      Send Response Data         <    Server(Establish)
```

### TLSv1.3 Session Resumption
Implementing `pre_shared_key` and `early_data` facilitates a TLS connection that essentially takes 0 RTT.

```bash
Client >  Say Hello, Key share, early data              >  Server
Client <  Server Hello, Key share, early data, finished <  Server
```