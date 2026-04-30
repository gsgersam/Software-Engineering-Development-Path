# ft_irc — Internet Relay Chat Server (C++98)
A group project focused on building an Internet Relay Chat server and understanding socket programming.

An event-driven IRC server for the 42 `ft_irc` project.
The implementation is based on a single non-blocking `poll()` loop and a strict parser/dispatcher/handler pipeline.

## Start Here

- Project server docs: [IRC/README.md](IRC/README.md)
- Developer docs (full technical flow): [documentation/DevDoc.md](documentation/DevDoc.md)
- Extended flowchart: [documentation/IRC_FLOWCHART.md](documentation/IRC_FLOWCHART.md)
- Deep audit report: [documentation/IRC_FLOW_AUDIT_REPORT.md](documentation/IRC_FLOW_AUDIT_REPORT.md)
- Official exercise notes: [documentation/Exercise_description.md](documentation/Exercise_description.md)

## Project Highlights

- Non-blocking sockets and one event loop (`poll`) for accept/read/write
- Multi-client support without thread-per-client model
- Robust IRC line parsing with trailing parameter handling
- Registration gate (`PASS` -> `NICK` -> `USER`) before command access
- Channel state management (members, operators, invite/key/limit/topic modes)
- Output buffering and `POLLOUT`-driven flush strategy

## Mandatory Scope

- Registration: `PASS`, `NICK`, `USER`
- Core: `PING`, `QUIT`, `PRIVMSG`
- Channels: `JOIN`, `PART`, `NAMES`, `LIST`
- Operator commands: `KICK`, `INVITE`, `TOPIC`, `MODE`
- Channel modes: `+i`, `+t`, `+k`, `+o`, `+l`

## Bonus Scope

- Bonus 1: DCC-aware `PRIVMSG` branch for file-transfer signaling
- Bonus 2: Bot helper in [IRC/bot/SimpleBot.cpp](IRC/bot/SimpleBot.cpp)

## Quick Run

```bash
cd IRC
make
./ircserv <port> <password>
```

Example:

```bash
./ircserv 6667 1234
```

Client test:

```bash
nc 127.0.0.1 6667
PASS 1234
NICK alice
USER alice 0 * :Alice
```

## Architecture Overview

- **Server engine**: owns sockets, pollfds, client/channel registries
- **Parser**: converts raw line into structured command
- **Dispatcher**: routes command and enforces registration gate
- **Handlers**: validate params/permissions and mutate logical state
- **Replies**: centralized formatting for numerics and protocol lines

## Reduced Flow (ASCII)

```text
Client Socket Event
        |
        v
  poll() in loop
   |      |
   |      +--> POLLOUT -> flush outBuffer with send()
   |
   +--> POLLIN -> recv() -> inBuffer
                    |
                    v
               extract IRC line
                    |
                    v
                 parseLine
                    |
                    v
                 dispatch
                    |
                    v
           handler updates state
           + queues replies/broadcast
                    |
                    v
             enable POLLOUT
```

## Full Flow (ASCII)

```text
+-------------------------------+
| Start: ./ircserv <p> <pass>   |
+-------------------------------+
               |
               v
+-------------------------------+
| Validate args + parsePort     |
+-------------------------------+
        | valid                 | invalid
        v                       v
+-----------------------+   +---------------------+
| Register SIGINT/TERM  |   | Print error + exit  |
+-----------------------+   +---------------------+
               |
               v
+-------------------------------+
| Create Server(port, password) |
+-------------------------------+
               |
               v
+-------------------------------+
| setupListenSocket()           |
| socket/bind/listen/nonblock   |
+-------------------------------+
               |
               v
+-------------------------------+
| addListenFdToPoll()           |
+-------------------------------+
               |
               v
+-------------------------------+
| run(): poll loop              |
+-------------------------------+
               |
               v
        +-------------------+
        | poll() has events?|
        +-------------------+
          | yes         | no/EINTR
          v             v
+------------------+   +------------------+
| iterate _pfds    |   | continue loop    |
+------------------+   +------------------+
          |
          v
+-------------------------+
| fd == listenFd ?        |
+-------------------------+
     | yes                      | no
     v                          v
+--------------------------+   +---------------------------+
| acceptNewClients()       |   | poll error/hup/nval ?     |
| accept until EAGAIN      |   +---------------------------+
+--------------------------+        | yes           | no
     |                               v              v
     |                        +----------------+  +------------------+
     |                        | disconnect(fd) |  | POLLIN on fd ?   |
     |                        +----------------+  +------------------+
     |                                              | yes       | no
     |                                              v           v
     |                                    +----------------+  +------------------+
     |                                    | handleRead(fd) |  | POLLOUT on fd ?  |
     |                                    +----------------+  +------------------+
     |                                              |             | yes      | no
     |                                              v             v          v
     |                                    +-----------------+ +----------------+
     |                                    | parse/dispatch  | | handleWrite(fd)|
     |                                    +-----------------+ +----------------+
     |                                              |             |
     |                                              +------->-----+
     |                                                        |
     +--------------------------------------------------------+
                                                              v
                                                +---------------------------+
                                                | checkTimeouts()           |
                                                +---------------------------+
                                                              |
                                                              v
                                                +---------------------------+
                                                | g_serverRunning == false? |
                                                +---------------------------+
                                                   | yes             | no
                                                   v                 v
                                      +-----------------------+   (loop)
                                      | Exit loop, cleanup    |
                                      | close sockets, return |
                                      +-----------------------+
```


Developed in collaboration with [Maria Florencia Calciati](https://github.com/mfcal223), [Ines passtor](https://github.com/inespastor) and [German Andres Sanin](https://github.com/gsgersam)
