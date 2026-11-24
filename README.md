# ETRAX network stack

A TCP/IP stack and HTTP server written from scratch in C++ for Axis' ETRAX (CRIS) embedded board - from raw Ethernet frames up to a web server with Basic authentication, with no OS network stack underneath.

Built in a computer networking course at Lund University (LTH), 2019, one layer at a time. The commit history follows that order.

## Layers

```
 HTTP/1.0 server     GET / POST, Basic auth, 401/404     src/net/http.cc
 File system         LZH archive compiled into image     src/app/fs.cc
 ───────────────────────────────────────────────────────────────────────
 Sockets             blocking read/write for threads     src/net/tcpsocket.cc
 TCP                 state machine, retransmission,      src/net/tcp.cc
                     receive-window-aware sending
 ───────────────────────────────────────────────────────────────────────
 IPv4 + ICMP         header checksums, echo reply        src/net/ip.cc, icmp.cc
 ARP                 replies for our address             src/net/arp.cc
 ───────────────────────────────────────────────────────────────────────
 LLC + Ethernet      frame rx/tx via controller buffers  src/net/llc.cc, ethernet.cc
 ───────────────────────────────────────────────────────────────────────
 Kernel (course)     threads, timers, semaphores, alloc  vendor/
```

Each layer decodes an `InPacket` and hands the payload to the layer above, and sends back down through the same chain. TCP connection states (LISTEN, SYN-RCVD, ESTABLISHED, CLOSE-WAIT, LAST-ACK, FIN-WAIT-1/2) are modelled with the State pattern.

## Layout

```
src/net/   protocol stack: Ethernet, LLC, ARP, IP, ICMP, TCP, sockets, HTTP
src/app/   board init, front-panel LEDs, in-memory file system
vendor/    course-provided kernel, boot code and runtime (unmodified)
www/       site content served by the board (wwwarc.lzh is the packed archive)
build/     source list and module definition
```

## Building

The image was built with the course's `genmake` tooling and CRIS cross-compiler on the LTH lab machines and flashed to the board, so it doesn't build standalone. `build/files` lists every source file that goes into the image.

## Credits

Written together with my lab partner Oskar. The kernel, boot code and runtime under `vendor/` were provided by the course, and `xprintf.c` is by D. R. Hipp.
