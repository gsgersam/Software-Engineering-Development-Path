# ft_irc Server — README

Event-driven IRC server in **C++98** using a **single non-blocking `poll()` loop**.

## Build and Run

```bash
make
./ircserv <port> <password>
```

Example:

```bash
./ircserv 6667 1234
```

## Quick Manual Test (Netcat)

```bash
nc 127.0.0.1 6667
PASS 1234
NICK alice
USER alice 0 * :Alice
```

Expected welcome numerics: `001`, `002`, `003`, `004`.

## Implemented Commands

- Registration: `PASS`, `NICK`, `USER`
- Core: `PING`, `QUIT`, `PRIVMSG`
- Channels: `JOIN`, `PART`, `NAMES`, `LIST`
- Operators: `KICK`, `INVITE`, `TOPIC`, `MODE`
- Channel modes: `+i`, `+t`, `+k`, `+o`, `+l`

## Bonus

- Bonus 1 (file transfer): DCC-aware `PRIVMSG` path for file-transfer signaling
- Bonus 2 (bot): [bot/SimpleBot.cpp](bot/SimpleBot.cpp)

## Reduced Runtime Flow (ASCII)

```text
poll() event
   |
   +--> listen fd -> accept() new client(s)
   |
   +--> client fd + POLLIN
   |        -> recv()
   |        -> parse line
   |        -> dispatch
   |        -> handler queues output
   |
   +--> client fd + POLLOUT
            -> send queued bytes
            -> disable POLLOUT when buffer is empty
```

## Full Technical Flow

See [documentation/DevDoc.md](../documentation/DevDoc.md) for full ASCII flow, registration gate, I/O pipeline, and bonus behavior.
