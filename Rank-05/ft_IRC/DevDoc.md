# ft_irc DevDoc (Developer Documentation)

Technical reference for execution flow, command routing, and bonus integration.

## Scope

### Mandatory

- Single event loop based on `poll()`
- Non-blocking sockets for listen and clients
- Registration gate (`PASS`, `NICK`, `USER`)
- IRC command dispatch to handlers
- Channel and operator features

### Bonus

- Bonus 1 (file transfer): DCC-aware branch in `PRIVMSG` for file-transfer signaling
- Bonus 2 (bot): [IRC/bot/SimpleBot.cpp](../IRC/bot/SimpleBot.cpp)

## Reduced Flow (ASCII)

```text
poll() loop
  |
  +--> accept new clients
  +--> read clients (POLLIN): recv -> parse -> dispatch -> queue
  +--> write clients (POLLOUT): send queued bytes -> clear POLLOUT
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

## Registration Gate

```text
Incoming command
      |
      v
Client registered?
  | yes                    | no
  v                        v
Dispatch known             Allow only PASS/NICK/USER/QUIT
commands                   others -> ERR_NOTREGISTERED (451)
```

## Input / Output Pipeline

```text
recv(fd) -> inBuffer -> extractLine -> parseLine -> dispatch -> handler
handler -> queueToClient/broadcast -> enable POLLOUT -> send(fd)
```

## Bonus 1 Branch (DCC-aware PRIVMSG)

```text
PRIVMSG text
   |
   v
CTCP wrapper?
   |
   +--> no  -> standard PRIVMSG
   +--> yes -> DCC SEND?
              |
              +--> no  -> standard CTCP forwarding
              +--> yes -> normalize IP format in DCC payload
                         -> queue to target
```

## Related Docs

- Extended flowchart: [documentation/IRC_FLOWCHART.md](IRC_FLOWCHART.md)
- Technical audit report: [documentation/IRC_FLOW_AUDIT_REPORT.md](IRC_FLOW_AUDIT_REPORT.md)
- Server usage guide: [IRC/README.md](../IRC/README.md)
