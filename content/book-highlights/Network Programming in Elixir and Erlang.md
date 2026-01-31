---
title: "Network Programming in Elixir and Erlang"
date: 2026-01-31
draft: false
tags:
  - book-highlights
---

# Part I: TCP

## Chapter 1: What Is Network Programming, Anyway?
- I watched [this](https://www.youtube.com/watch?index=9&list=PLKBMoE8mCkXgd0VNxA4y0DdKRJ_4EUPyN&v=9UFeQ11soQg) video as a helpful intro
	- references [this](https://ferd.ca/it-s-about-the-guarantees.html) blog post
- this means software that communicates with other software, potentially on different machines, through a network
- Distribution in Erlang is fundamentally based on networking, since Erlang nodes in a cluster generally communicate with each other via TCP. However, the distribution aspect is orthogonal to networking itself
- Erlang’s actor model, lightweight processes, and message passing are a perfect fit for network applications, which are often made of separate, independent clients and servers (processes) passing information around (messages) in an asynchronous, reactive way

```
+-------------------+         +-------------------+
|                   |         |                   |
|                   |         |                   |
|   Erlang Process  |         |   Erlang Process  |
|         ^         |         |         ^         |
|         |         |         |         |         |
|         |         |         |         |         |
|         |         |         |         |         |
|         v         |         |         v         |
| Network Connection|<------->| Network Connection|
|                   |  Data   |                   |
+-------------------+         +-------------------+
```
- On the BEAM processes are as cheap as it gets: so much so that most Erlang or Elixir net- work applications allocate one process for each network connection
	- These processes have isolated memory and isolated failure semantics. If a process crashes, it only brings down the connection that it is managing, without affecting other connections
	- Isolated memory makes garbage collection fast and distributed across processes

# Part 1: TCP

## Chapter 2: TCP: Exploring the Basics
- TCP is the most ubiquitous transport-layer protocol for network applications.
	- most interactions on networks can be modeled as a server and a client talking to each other.
	- a server has to actively listen for incoming connections on a specific address and port combination. A TCP client can then establish a connection to that server to exchange data with it
- When a client connects to a server, a connection gets opened on each side. This connection is persistent and remains opened until one of the two sides terminates it. This is possible because TCP is a stateful protocol. Clients and servers have to store some state in the connection. TCP uses this state to keep track of packet order and deal with duplicated packets
- TCP connections are always represented by an operating-system-level abstraction, the socket
	- Sockets are the mechanism that the OS provides to give programs access to the network
	- a socket is a data structure that contains a bunch of data related to the connection
- the socket abstraction is represented by a port. A port is an Erlang primitive data type used to communicate with external programs
- TCP is also known as TCP/IP
	- Its job is to route packets from sources to destination
		- an address and a port
			- ports are a way to route packets to different programs within the same host
- Client: `:gen_tcp.connect(~c"tcpbin.com", 4242, [mode: :binary], 5000)`
	- passive: `:gen_tcp.recv(socket, 0, 5000)`
		- any process can call  `:gen_tcp.recv/3`
			- only one process can be receiving data from the socket at any given time
	- active: `flush()`
- :gen_tcp sockets can generally be in one of two modes: active mode or passive mode. By default, they start in active mode. In this mode, :gen_tcp delivers all the data that the socket receives
	- The :gen_tcp API defines three possible messages
	- These two modes (active/passive) have different properties intended for different use cases. For example, active sockets are generally preferred when you want to avoid blocking the receiving process, such as when said process is an OTP process like a GenServer. Passive mode, however, can be more efficient and gives you finer control over how much data to receive
	- Every :gen_tcp socket has a controlling process. This is a BEAM process that is “responsible” for the socket itself
		- The socket is linked to its controlling process, which means that if the controlling process exits, then the BEAM automatically shuts the socket down and cleans things up
			- The controlling process of a socket also has another property: it’s the only process that can receive data from the socket when the socket is in active mode
- Server: `:gen_tcp.listen(4000, [mode: :binary, active: true])`

## Chapter 3: Designing a Chat Protocol and Its TCP Server
- A binary protocol is generally characterized by encoding information in bytes or bits
- Textual protocols are protocols that are meant to be readable by machines and humans alike. Bytes of information are interpreted through a specified encoding, most commonly ASCII. One of the most famous examples of such a protocol is JSON.
- Which kind of protocol should you choose, textual or binary? As always, the best answer we can give to this sort of question is: “it depends.” One of the main factors to take into consideration is performance and size.
	- Well-designed binary protocols are inevitably more space efficient when seri- alizing data, since they can use a granularity of a single bit to encode information
	- The downside of binary protocols is that they’re mostly unintelligible for humans
	- Endiannessa is the order in which bytes are interpreted
- When designing a protocol, start from the requirements of the protocol
- It’s common to prefix the hexadecimal value with the symbols 0x
- encode strings of text. A common way to do that in binary protocols is to use a few bytes to encode the length of the string, followed by the contents of the string itself
- register messages: unidirectional messages that clients must send as the first message after establishing a connection
- broadcast messages: bidirectional messages. Their type byte has value 0x02, and it’s followed by a string representing the “from” username of the message, and then one more string with the contents of the message itself
- It’s almost always a good idea to keep the protocol code separate from the networking code when possible. Protocol code is often stateless and free of side effects


## Chapter 4: Scaling TCP on the Server Side
- order matters in an application's children list
- When designing non-trivial supervision trees like the one we’re
dealing with here, I always sketch the supervision tree out before
writing any code. Having a visual reference makes things significantly easier for me. This is especially true because it’s harder to “see” the supervision tree structure by just looking at code and files. I recommend you always do the same as well.
	- verify ypur sketch matches output from `:observer.start()`

**Meeting Notes**
- use `defstruct` is GenServers for state
- Registry: state managed for free
- Task: fire and forget or else use Oban


## Chapter 5: Building TCP Clients
- We could move forward in this direction and wrap this whole interaction in a function. Users of our Redis client would call this function to send a command and get a response. However, we’d run into quite a few of issues right away. The biggest issue would be that, for most use cases, this approach is awfully inefficient. Opening a new TCP client socket every time we want to exchange some information with a server goes directly against one of the fundamental properties that characterize TCP itself: TCP is a stateful, connection-oriented protocol. It’s built from the ground up to work with longer-lived connections, which only perform the initial handshake once and can then exchange data back and forth. TCP sockets also take up operating-system resources, so opening new sockets is not “free”. Last but not least, the TCP handshake itself has a non-negligible cost in terms of computation and network round-trips
	- Because of all these reasons, what we would ideally do is reuse TCP client sockets for multiple exchanges with the server. To do that, we need to “store” the TCP socket somewhere while it’s not being used for requests. On the BEAM, that somewhere is a process
	- We want the socket to be in active mode because we want our client to receive :tcp_closed and :tcp_error messages in case the connection is interrupted. Since that is bound to happen at some point
- The main advantage of this approach to TCP clients on the BEAM is that we move the TCP socket around instead of moving the data around. On the BEAM, sending a message from a process to another generally means copying the data being sent from the memory of the sending process to the memory of the receiving process. The only case when this is not true is for binaries larger than 64 bytes, which live in a common memory space. This is a desirable property in a virtual machine like ours, because it allows processes to be completely isolated and allows the VM to garbage-collect processes individually. However, a TCP client like the one we built is mostly sending messages to and from caller processes, so the memory overhead of always copying those messages can be impactful. Moving the socket around, instead, means that callers can read off of and write to the OS socket buffer directly, without copying data within the BEAM.
	- Small binaries below 64 bytes in size are stored in the process heap (heap binaries), but binaries larger than that are stored in a shared memory area.
	- if you expect most messages with a TCP server to be large binaries, then you could avoid passing the socket around, since the binaries will likely be refc binaries anyway.
- This approach has one drawback when it comes to data though: our client is not taking advantage of TCP multiplexing. Multiplexing is a property of TCP which essentially means that a TCP socket can receive and send data simultaneously
- As a general rule, whenever your system has a queue, you’ll likely want to enforce an upper bound on the size of that queue

## Chapter 6: Scaling and Optimizing TCP Clients
- Poolboy is a great tool to reach for when you want to scale something up by adding “copies” of the same worker process
	- Poolboy is especially useful in cases where the workers you want to pool are not really built for multiple concurrent caller requests
	- used to pool processes
- While our Redis client is a process, what it’s really doing is mostly wrappinga TCP socket and handing it around to other processes. We can think of that TCP socket as a resource
	- NimblePool is a small open-source library focused on writing pools of resources. The idea behind the library is this: if you want to pool resources, it might be enough to have a single process managing all of those resources and handing them out to callers. This idea works in many cases, because the bulk of the work is done in the callers
- TCP is a full-duplex protocol. This means that the protocol is bidirectional, that is, data can flow both ways. It also means that data can flow both ways at the same time
	- a state machine can take advantage of this
	- TCP clients can usually be modeled as FMSs (finite state machines) with just two states: connected and disconnected
