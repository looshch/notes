# Notes On General CS Topics
* [Process](#process)
  * [Thread](#thread)
* [Network](#network)
  * [OSI](#osi)
  * [IP](#ip)
    * [TCP](#tcp)
    * [UDP](#udp)
    * [TLS](#tls)
    * [HTTP](#http)
    * [Server-Sent Events](#server-sent-events)
    * [Latency](#latency)
* [Cache](#cache)

## Process
* a process is just an instance of an executing program, including the current
  values of the program counter, registers, and variables. Conceptually, each
  process has its own virtual CPU

* one way of looking at a process is that it is a way to group related resources
  together. A process has an address space containing program text and data, as
  well as other resources. These resources may include open files, child
  processes, pending alarms, signal handlers, accounting information, and more.
  By putting them together in the form of a process, they can be managed more
  easily

* to implement the process model, the operating system maintains a table (an
  array of structures), called the process table, with one entry per process.
  (Some authors call these entries process control blocks.) This entry contains
  important information about the process’ state, including its program counter,
  stack pointer, memory allocation, the status of its open files, its accounting
  and scheduling information, and everything else about the process that must be
  saved when the process is switched from running to ready or blocked state so
  that it can be restarted later as if it had never been stopped

* after the fork, the two processes, the parent and the child, have the same
  memory image, the same environment strings, and the same open files. In UNIX,
  the child’s initial address space is a copy of the parent’s, but there are
  definitely two distinct address spaces involved; no writable memory is shared.
  Some UNIX implementations share the program text between the two since that
  cannot be modified. Alternatively, the child may share all of the parent’s
  memory, but in that case the memory is shared copy-on-write, which means that
  whenever either of the two wants to modify part of the memory, that chunk of
  memory is explicitly copied first to make sure the modification occurs in a
  private memory area. Again, no writable memory is shared. It is, however,
  possible for a newly created process to share some of its creator’s other
  resources, such as open files

* Coffman et al. showed that four conditions must hold for there to be a
  (resource) deadlock:
  * mutual exclusion condition. Each resource is either currently assigned
    to exactly one process or is available
  * hold-and-wait condition. Processes currently holding resources that were
    granted earlier can request new resources
  * no-preemption condition. Resources previously granted cannot be forcibly
    taken away from a process. They must be explicitly released by the process
    holding them
  * circular wait condition. There must be a circular list of two or more
    processes, each of which is waiting for a resource held by the next member
    of the chain
  All four of these conditions must be present for a resource deadlock to occur.
  If one of them is absent, no resource deadlock is possible

* ### Thread
  * what threads add to the process model is to allow multiple executions to
    take place in the same process environment, to a large degree independent
    of one another. Having multiple threads running in parallel in one process
    is analogous to having multiple processes running in parallel in one
    computer. In the former case, the threads share an address space and other
    resources. In the latter case, processes share physical memory, disks,
    printers, and other resources. Because threads have some of the properties
    of processes, they are sometimes called lightweight processes. The term
    multithreading is also used to describe the situation of allowing multiple
    threads in the same process

  * with threads we add a new element: the ability for the parallel entities to
    share an address space and all of its data among themselves. This ability
    is essential for certain applications, which is why having multiple
    processes (with their separate address spaces) will not work. A second
    argument for having threads is that since they are lighter weight than
    processes, they are easier (i.e., faster) to create and destroy than
    processes. In many systems, creating a thread goes 10–100 times faster
    than creating a process. Threads yield no performance gain when all of
    them are CPU bound, but when there is substantial computing and also
    substantial I/O, having threads allows these activities to overlap, thus
    speeding up the application

  * the thread has a program counter that keeps track of which instruction to
    execute next. It has registers, which hold its current working variables.
    It has a stack, which contains the execution history, with one frame for
    each procedure called but not yet returned from. Although a thread must
    execute in some process, the thread and its process are different concepts
    and can be treated separately. Processes are used to group resources
    together; threads are the entities scheduled for execution on the CPU

  * each thread’s stack contains one frame for each procedure called but not
    yet returned from. This frame contains the procedure’s local variables and
    the return address to use when the procedure call has finished

  * associated with each I/O class is a location (typically at a fixed lo cation
    near the bottom of memory) called the interrupt vector. It contains the
    address of the interrupt service procedure. Suppose that user process 3 is
    running when a disk interrupt happens. User process 3’s program counter,
    program status word, and sometimes one or more registers are pushed onto the
    (current) stack by the interrupt hardware. The computer then jumps to the
    address specified in the interrupt vector. That is all the hardware does.
    From here on, it is up to the software, in particular, the interrupt service
    procedure. All interrupts start by saving the registers, often in the
    process table entry for the current process. Then the information pushed
    onto the stack by the interrupt is removed and the stack pointer is set to
    point to a temporary stack used by the process handler. When this routine is
    finished, it calls a C procedure to do the rest of the work for this
    specific interrupt type. When it has done its job, possibly making some
    process now ready, the scheduler is called to see who to run next. After
    that, control is passed back to the assembly-language code to load up the
    registers and memory map for the now-current process and start it running.
    A process may be interrupted thousands of times during its execution, but
    the key idea is that after each interrupt the interrupted process returns
    to precisely the same state it was in before the interrupt occurred

## Network
* a gateway is a generic term for an entity used to connect two or more
  networks. A repeater operates at the physical level copies the information
  from one subnet to another. A bridge operates at the data link layer level and
  copies frames between networks. A router operates at the network level and not
  only moves information between networks but also decides on the route

* the communication between layers in either the OSI or the TCP/IP stacks is
  done by sending packets of data from one layer to the next, and then
  eventually across the network. Each layer has administrative information that
  it has to keep about its own layer. It does this by adding header information
  to the packet it receives from the layer above, as the packet passes down. On
  the receiving side, these headers are removed as the packet moves up

* ### OSI
  * 1. `Physical_____down` layer conveys the bit stream using electrical,
       optical, or radio technologies
    1. `Data Link_____|__` layer puts the information packets into network
       frames for transmission across the physical layer, and back into
       information packets
    1. `Network_______|__` layer provides switching and routing technologies
    1. `Transport_____|__` layer provides transparent transfer of data between
       end systems and is responsible for end-to-end error recovery and flow
       control
    1. `Session_______|__` layer establishes, manages and terminates connections
       between applications
    1. `Presentation_\|/_` layer provides independence from differences in data
       representation (e.g. encryption)
    1. `Application__top_` layer supports application and end-user processes

  * although it was never properly implemented, the OSI (Open Systems
    Interconnect) protocol has been a major influence in ways of talking about
    and influencing distributed systems design

* ### IP
  * TCP/IP
    1. `HW Interface_down` as Physical and Data Link
    1. `IP____________|__` as Network
    1. `TCP/UDP______\|/_` as Transport
    1. `Application__top_` as Session, Presentation and Application

  * routers differ from other computing devices in that they have at least two
    IP address: a public one and a private one. The public side of a router is
    visible on the Internet and is also referred to as the WAN or Wide Area
    Network side of the router. The router has no control over the public IP
    address, it is assigned by the ISP and it is not a secret and there are many
    websites that display it. In contrast, the router has total control over the
    LAN or Local Area Network side IP addresses, both for itself and for all the
    computing devices that connect to it

  * the range of allowable LAN side IP addresses is called a subnet (as in
    sub-network, as in, only use these few numbers of all the billions of
    possible numbers). A very common subnet (range of numbers) is the numbers
    that start with 192.168.1 and only vary in the fourth/last number. This is
    often written as 192.168.1.x where the x is a placeholder for all the
    possible numbers in the fourth position (0 to 255)

  * #### TCP
  transmission control protocol, RFC 793

  * guarantees that all bytes sent will be identical with bytes received and
    that they will arrive in the same order to the client

  * connections begin with a three-way handshake
    * SYN — client picks a random sequence number x and sends a SYN packet,
      which may also include additional TCP flags and options
    * SYN ACK — server increments x by one, picks own random sequence number y,
      appends its own set of flags and options, and dispatches the response
    * ACK — client increments both x and y by one and completes the handshake by
      dispatching the last ACK packet in the handshake
    the client can send a data packet immediately after the ACK packet, and the
    server must wait for the ACK before it can dispatch any data

  * new connections are expensive to create. Connection reuse is a critical
    optimization for any application running over TCP
    * TCP Fast Open (TFO) is a mechanism that aims to reduce the latency penalty
      imposed on new TCP connections. It works only in certain cases: there are
      limits on the maximum size of the data payload within the SYN packet, only
      certain types of HTTP requests can be sent, and it works only for repeat
      connections due to a requirement for a cryptographic cookie

  * flow control is a mechanism to prevent the sender from overwhelming the
    receiver with data it may not be able to process. Each side of the TCP
    connection advertises its own receive window (rwnd), which communicates the
    size of the available buffer space to hold the incoming data. Each ACK
    packet carries the latest rwnd value for each side
    * on macOS, defautl value is 3 (sysctl net.inet.tcp.win_scale_factor). It’s
      suggested to use 8 (???)

  * congestion window size (cwnd) — sender-side limit on the amount of data the
    sender can have in flight before receiving an acknowledgment (ACK) from the
    client

  * the only way to estimate the available capacity between the client and the
    server is to measure it by exchanging data, and this is precisely what
    slow-start is designed to do. To start, the server initializes a new
    congestion window (cwnd) variable per TCP connection and sets its initial
    value to a conservative, system-specified value (initcwnd). The cwnd
    variable is not advertised or exchanged between the sender and receiver. The
    maximum amount of data in flight (not ACKed) between the client and the
    server is the minimum of the rwnd and cwnd variables. To determine optimal
    values for congestion window sizes, client and server start slow and grow
    the window size as the packets are acknowledged. The server can send up to
    four network segments to the client, at which point it must stop and wait
    for an acknowledgment. Then, for every received ACK, the slow-start
    algorithm indicates that the server can increment its cwnd window size by
    one segment —for every ACKed packet, two new packets can be sent. This phase
    of the TCP connection is commonly known as the ‘exponential growth’
    algorithm. For many HTTP connections, which are often short and bursty, it
    is not unusual for the request to terminate before the maximum window size
    is reached. In addition to regulating the transmission rate of new
    connections, TCP also implements a slow-start restart (SSR) mechanism, which
    resets the congestion window of a connection after it has been idle for a
    defined period of time. SSR can have a significant impact on performance of
    long-lived TCP connections that may idle for bursts of time, so it is
    recommended to disable SSR on the server

  * increasing the initial cwnd size on the server to the new RFC 6928 value of
    10 segments is one of the simplest ways to improve performance for all users
    and all applications running over TCP

  * perfomance checklist
    * server kernel is upgraded to latest version
    * cwnd size is set to 10
    * slow-start after idle is disabled
    * that window scaling is enabled
    * data compression
    * TCP connections are reused if possible

  * #### UDP
  user datagram (null) protocol, RFC 768

  * datagram is a self-contained, independent entity of data carrying sufficient
    information to be routed from the source to the destination nodes without
    reliance on earlier exchanges between the nodes and the transporting network

  * the words datagram and packet are often used interchangeably, but there are
    some nuances. While the term ‘packet’ applies to any formatted block of
    data, the term ‘datagram’ is often reserved for packets delivered via an
    unreliable service

  * #### TLS
  transport layer security, RFC 2246

  * connections begin with a handshake
    * TCP three-way handshake
    * client sends a number of specifications in plain text, such as the version
      of the TLS protocol it is running, the list of supported ciphersuites, and
      other TLS options it may want to use
    * server picks the TLS protocol version for further communication, decides
      on a ciphersuite from the list provided by the client, attaches its
      certificate, and sends the response back to the client. Optionally, the
      server can also send a request for the client’s certificate and parameters
      for other TLS extensions
    * the client then generates a new symmetric key, encrypts it with the
      server’s public key, and tells the server to switch to encrypted
      communication going forward. Up until now, all the data has been exchanged
      in clear text with the exception of the new symmetric key that is
      encrypted with the server’s public key
    * the server decrypts the symmetric key sent by the client, checks message
      integrity by verifying the MAC (a message authentication code), and
      returns an encrypted ‘Finished’ message back to the client
    * the client decrypts the message with the symmetric key it generated
      earlier, verifies the MAC, and if all is well, then the tunnel is
      established and application data can now be sent

  * the MAC algorithm is a one-way cryptographic hash function (effectively a
    checksum), the keys to which are negotiated by both connection peers.
    Whenever a TLS record is sent, a MAC value is generated and appended for
    that message, and the receiver is then able to compute and verify the sent
    MAC value to ensure message integrity and authenticity

  * public-key cryptography is used only during session setup of the TLS tunnel.
    The server provides its public key to the client, and then the client
    generates a symmetric key, which it encrypts with the server’s public key,
    and returns the encrypted symmetric key to the server. Finally, the server
    can decrypt the sent symmetric key with its private key. Symmetric key
    cryptography, which uses the shared secret key generated by the client, is
    then used for all further communication between the client and the server.
    This is done, in large part, to improve performance—public key cryptography
    is much more computationally expensive

  * session identifiers — the server could then maintain a cache of session IDs
    and the negotiated session parameters for each peer. In turn, the client
    could then also store the session ID information and include the ID in the
    ‘ClientHello’ message for a subsequent session, which serves as an
    indication to the server that the client still remembers the negotiated
    cipher suite and keys from previous handshake and is able to reuse them.
    Assuming both the client and the server are able to find the shared session
    ID parameters in their respective caches, then an abbreviated handshake can
    take place (four-way). Most modern browsers intentionally wait for the first
    TLS connection to complete before opening new connections to the same
    server: subsequent TLS connections can reuse the SSL session parameters to
    avoid the costly handshake. One of the practical limitations of the Session
    Identifiers mechanism is the requirement for the server to create and
    maintain a session cache for every client. This results in several problems
    on the server, which may see tens of thousands or even millions of unique
    connections every day: consumed memory for every open TLS connection, a
    requirement for session ID cache and eviction policies, and nontrivial
    deployment challenges for popular sites with many servers, which should,
    ideally, use a shared TLS session cache for best performance. To address
    this concern for server-side deployment of TLS session caches, the ‘Session
    Ticket’ replacement mechanism was introduced, which removes the requirement
    for the server to keep per-client session state. Instead, if the client
    indicated that it supports Session Tickets, in the last exchange of the full
    TLS handshake, the server can include a New Session Ticket record, which
    includes all of the session data encrypted with a secret key known only by
    the server. Thus, all session data is stored only on the client, but the
    ticket is still safe because it is encrypted with a key known only by the
    server. The session identifiers and session ticket mechanisms are
    respectively commonly referred to as session caching and stateless
    resumption mechanisms

  * certificate Revocation List (CRL) is defined by RFC 5280 and specifies a
    simple mechanism to check the status of every certificate: each certificate
    authority maintains and periodically publishes a list of revoked certificate
    serial numbers. CRL mechanism may be insufficient in the following
    instances:
    * the growing number of revocations means that the CRL list will
      only get longer, and each client must retrieve the entire list of serial
      numbers
    * there is no mechanism for instant notification of certificate
      revocation—if the CRL was cached by the client before the certificate was
      revoked, then the CRL will deem the revoked certificate valid until the
      cache expires
    to address some of the limitations of the CRL mechanism, the Online
    Certificate Status Protocol (OCSP) was introduced by RFC 2560, which
    provides a mechanism to perform a real-time check for status of the
    certificate. The client must block on OCSP requests before proceeding with
    the navigation. CRL and OCSP mechanisms are complementary, and most
    certificates will provide instructions and endpoints for both. Some browsers
    distribute their own CRL lists, others fetch and cache the CRL files from
    the CAs. Similarly, some browsers will perform the real-time OCSP check but
    will differ in their behavior if the OCSP request fails

  * the TLS record protocol is responsible for identifying different types of
    messages (handshake, alert, or data via the ‘Content Type’ field), as well
    as securing and verifying the integrity of each message. A typical workflow
    for delivering application data is as follows:
    * record protocol receives application data
    * received data is divided into blocks: maximum of 16 KB per record
    * application data is optionally compressed
    * message authentication code (MAC) or HMAC is added
    * data is encrypted using the negotiated cipher
    on the receiving end, the same workflow, but in reverse, is applied by the
    peer. Each record contains a 5-byte header, a MAC (up to 20 bytes for SSLv3,
    TLS 1.0, TLS 1.1, and up to 32 bytes for TLS 1.2), and padding if a block
    cipher is used. Picking the right record size for an application, if you
    have the ability to do so, can be an important optimization. Small records
    incur a larger overhead due to record framing, whereas large records will
    have to be delivered and reassembled by the TCP layer before they can be
    processed by the TLS layer and delivered to your application

  * you should disable TLS compression on your server for several reasons:
    * the ‘CRIME’ attack, leverages TLS compression to recover secret
      authentication cookies and allows the attacker to perform session
      hijacking * transport-level TLS compression is not content aware and will
      end up attempting to recompress already compressed data (images, video,
      etc.)
  * verifying the chain of trust requires that the browser traverse the chain,
    starting from the site certificate, and recursively verifying the
    certificate of the parent until it reaches a trusted root. Hence, the first
    optimization you should make is to verify that the server does not forget to
    include all the intermediate certificates when the handshake is performed

  * perfomance checklist
    * enable and configure session caching and stateless resumption
    * configure your TLS record size to fit into a single TCP segment
    * ensure that your certificate chain does not overflow the initial
      congestion window
    * disable TLS compression on your server.
    * configure SNI support on your server.
    * configure OCSP stapling on your server.
    * append HTTP Strict Transport Security header.

  * #### HTTP
  * all HTTP 2.0 communication is performed within a connection that can carry
    any number of bidirectional streams. In turn, each stream communicates in
    messages, which consist of one or multiple frames, each of which may be
    interleaved and then reassembled via the embedded stream identifier in the
    header of each individual frame
    * all communication is performed with a single TCP connection
    * the stream is a virtual channel within a connection, which carries
      bidirectional messages. Each stream has a unique integer identifier (1, 2,
      ..., N)
    * the message is a logical HTTP message, such as a request, or response,
      which consists of one or more frames
    * the frame is the smallest unit of communication, which carries a specific
      type of data—e.g., HTTP headers, payload, and so on

  * once an HTTP message can be split into many individual frames, the exact
    order in which the frames are interleaved and delivered can be optimized to
    further improve the performance of our applications. To facilitate this,
    each stream can be assigned a 31-bit priority value:
    * 0 represents the highest priority stream
    * 231–1 represents the lowest priority stream

  * HTTP 2.0 is able to to send multiple replies from the server to a client
    for a single request

  * #### Server-Sent Events
  server-sent events enables efficient server-to-client streaming of text-based
  event data

  * #### Latency
  latency, not bandwidth, is the performance bottleneck for most websites

  * common types of delays
    * propagation — amount of time required for a message to travel from the
      sender to receiver, which is a function of distance over speed with which
      the signal propagates
    * transmission — amount of time required to push all the packet’s bits into
      the link, which is a function of the packet’s length and data rate of the
      link
    * processing — amount of time required to process the packet header,
      check for bit-level errors, and determine the packet’s destination
    * queuing — amount of time the incoming packet is waiting in the queue until
      it can be processed
## Cache
  * * cache hit — data served from it
    * cache miss — data not served from it
