# Full Flow Technical Audit Report — ft_irc

## 1) Executive Summary

This document describes the real execution flow of the `ft_irc` project, from server startup to client disconnection and final process shutdown.

It covers:
- Server initialization.
- Event loop with `poll`.
- Input pipeline: `recv -> inBuffer -> extractLine -> parseLine -> dispatch -> handler`.
- Output pipeline: `queue -> POLLOUT -> send`.
- Internal state of `Server`, `Client`, and `Channel`.
- Implemented IRC commands (`PASS`, `NICK`, `USER`, `PING`, `PRIVMSG`, `JOIN`, `MODE`, `TOPIC`, `INVITE`, `LIST`, `NAMES`, `PART`, `KICK`, `QUIT`).
- Chronological multi-user simulation.
- Clean shutdown by signal and timeout.

---

## 2) Global Architecture

### 2.1 Main Modules

- **Network engine (`Server`)**
  - Owns sockets, `pollfd`, client map, and channel map.
  - Performs `accept/recv/send/close`.
  - Coordinates disconnects and cleanup.

- **Parser (`Parser`)**
  - Converts one IRC line into a structured `Command`.
  - Does not perform networking and does not mutate global state.

- **Dispatcher (`Dispatcher`)**
  - Routes `Command.name` to the correct handler.
  - Enforces registration rules before allowing commands.

- **Handlers (`handler*.cpp`)**
  - Validate parameters, permissions, and state.
  - Update logical state (`Client`, `Channel`, server registries).
  - Queue outgoing messages.

- **Domain model (`Client`, `Channel`)**
  - `Client`: identity, buffers, registration flags, joined channels, quit state.
  - `Channel`: members, operators, topic, `i/t/k/l` modes, invite list.

- **Replies (`Replies`)**
  - Builds RFC numeric replies and prefixed protocol messages.

### 2.2 Key Design Principle

Handlers **do not execute network syscalls** (`send/recv/poll`).
They only queue data (`queueToClient` / `broadcastToChannel`), and the engine flushes buffers when `POLLOUT` is available.

---

## 3) Server Startup

## 3.1 Entry through `main`

Sequence:
1. Argument validation: `./ircserv <port> <password>`.
2. Port validation with `parsePort` (range 1024..65535).
3. Signal registration (`SIGINT`, `SIGTERM`).
4. Construct `Server(port, password)`.
5. Blocking call to `server.run()`.

Signal effect:
- Signal handler sets `g_serverRunning = false`.
- Main loop exits cleanly.

## 3.2 Server socket setup

`setupListenSocket()` performs:
1. `socket(AF_INET, SOCK_STREAM, 0)`.
2. `setsockopt(SO_REUSEADDR)`.
3. `fcntl(..., O_NONBLOCK)`.
4. `bind(INADDR_ANY, port)`.
5. `listen(backlog=128)`.

Then `addListenFdToPoll()` adds `listenFd` to `_pfds` with `POLLIN`.

---

## 4) Event Loop (`poll`) and connection lifecycle

## 4.1 General structure

Inside `Server::run()`:
- `poll(_pfds, -1)` waits indefinitely.
- Iterates through fds with active `revents`.

Logical order for each client fd:
1. If `POLLERR | POLLHUP | POLLNVAL` -> `disconnectClient(fd)`.
2. If `POLLIN` -> `handleClientRead(fd)`.
3. If `POLLOUT` -> `handleClientWrite(fd)`.
4. If client is `isQuitting` and `outBuffer` is empty -> `disconnectClient(fd)`.
5. End of round: `checkTimeouts()`.

## 4.2 Accepting new clients

`acceptNewClients()`:
- Calls `accept` in a loop until `EAGAIN/EWOULDBLOCK`.
- Marks client fd as non-blocking.
- Inserts `Client(fd)` into `_clientsByFd`.
- Adds fd to `_pfds` with `POLLIN`.

## 4.3 Client read path

`handleClientRead(fd)`:
1. `recv` into a temporary 4096-byte buffer.
2. If `n==0`: peer closed connection -> `disconnectClient`.
3. Fatal error: disconnect.
4. `EAGAIN/EWOULDBLOCK`: return without failure.
5. `client.inBuffer += bytes`.
6. While complete lines exist:
   - `extractLine` splits at `\n` and strips trailing `\r`.
   - `Parser::parseLine` builds a `Command`.
   - `Dispatcher::handle` invokes the handler.

## 4.4 Client write path

`handleClientWrite(fd)`:
1. If `outBuffer` is empty -> disable `POLLOUT`.
2. `send` pending bytes.
3. On partial send, erase only sent bytes.
4. If buffer becomes empty, disable `POLLOUT`.
5. If client was quitting and buffer is empty -> disconnect.

---

## 5) Parsing in detail

`Parser::parseLine` processes one already-extracted IRC line:

1. `cleanLine`: trims leading/trailing CR/LF and leading spaces.
2. If line starts with `:`, parse `prefix`.
3. Parse `name` and uppercase it.
4. Parse params:
   - regular space-separated tokens;
   - if token starts with `:`, concatenate the remainder as one trailing parameter.

Examples:
- `NICK ana` -> `name=NICK, params=[ana]`.
- `PRIVMSG #c :hello world` -> `params=[#c, hello world]`.

---

## 6) Dispatcher and registration control

`Dispatcher::handle` enforces rules before calling handlers:

- If client is not registered:
  - only allows `PASS`, `NICK`, `USER`, `QUIT`.
  - if `passOk=false`, blocks `NICK` and `USER` with `451`.
- Unknown command name -> `421 Unknown command`.

Registered handlers:
- PASS, NICK, USER, PING, QUIT,
- PRIVMSG, JOIN, KICK, INVITE, TOPIC, MODE,
- PART, LIST, NAMES.

---

## 7) Internal state: Server / Client / Channel

## 7.1 Server

Core structures:
- `_clientsByFd: map<int, Client>`.
- `_fdByNick: map<string,int>`.
- `_channelsByName: map<string, Channel>`.
- `_pfds: vector<pollfd>`.

Critical operations:
- `bindNick/unbindNick/isNickTaken`.
- `getOrCreateChannel/getChannel`.
- `removeClientFromChannel/partClientFromChannel/removeChannelIfEmpty`.
- `ensureChannelHasOperator` (promotes one member if no operators remain).
- `notifyClientExit` (QUIT broadcast to shared members).

## 7.2 Client

Relevant fields and flags:
- Buffers: `_inBuffer`, `_outBuffer`.
- Registration: `_passOk`, `_hasNick`, `_hasUser`, `_registered`.
- Identity: `_nick`, `_user`, `_realname`.
- Quit state: `_isQuitting`, `_quitMsg`.
- Joined channels: `_channels`.
- Timeout tracking: `_lastActivity`.

## 7.3 Channel

Per-channel state:
- Members `_members`.
- Operators `_operators`.
- Topic `_topic`.
- Modes:
  - `_inviteOnly` (`+i`)
  - `_topicOpsOnly` (`+t`)
  - `_hasKey/_key` (`+k`)
  - `_hasLimit/_limit` (`+l`)
- Invited clients `_invited`.

---

## 8) Commands and exact per-handler flow

## 8.1 Registration

### PASS
- Rejects if already registered (`462`).
- Requires one parameter (`461`).
- Wrong password -> `464` and `passOk=false`.
- Correct password -> `passOk=true`.

### NICK
- Requires valid PASS first (`451`).
- Missing nick -> `431`.
- Invalid nick -> `432`.
- Nick already in use -> `433`.
- If nick changes and client is already registered:
  - broadcasts `NICK` to users sharing channels.
- Safely rebinds in `_fdByNick`.

### USER
- Already registered -> `462`.
- Missing params -> `461`.
- Sets `user` and optional `realname`.

### Registration completion
- `Client::tryCompleteRegistration` sets `registered` when PASS+NICK+USER are present.
- `Replies::maybeSendWelcome` sends welcome block (001..004 + extra message).

## 8.2 Keepalive

### PING
- Missing token -> `461`.
- Replies with `PONG` carrying the same token.

## 8.3 Messaging

### PRIVMSG
- Requires registered client.
- Validation:
  - missing target -> `411`.
  - missing text -> `412`.
- If target is channel `#`:
  - channel does not exist -> `403`.
  - sender not a member -> `404`.
  - if valid, broadcast to channel except sender.
- If target is nick:
  - target missing -> `401`.
  - target exists -> queue direct message.
- Bonus: detects CTCP DCC SEND and adapts IP format when needed.

## 8.4 Channels

### JOIN
- Special case `JOIN 0`: leaves all channels for that client.
- Validates channel name (`#...`) and params.
- Creates channel if needed.
- Restrictions:
  - `+k` requires correct key (`475`).
  - `+i` requires invite (`473`).
  - `+l` requires free slot (`471`).
- On success:
  - adds member in both channel and client state.
  - first member becomes operator.
  - clears used invite.
  - broadcasts `JOIN`.
  - sends topic (if any) and `NAMES`.

### PART
- Accepts list `#a,#b,#c`.
- For each channel:
  - validates name and existence (`403`).
  - validates membership (`442`).
  - emits `PART` and removes member.
- If channel becomes empty, deletes it.
- If channel remains but has no operators, auto-promotes one member.

### KICK
- Requires channel + target params (`461`).
- Channel missing -> `403`.
- Requester not on channel -> `442`.
- Requester is not op -> `482`.
- Target nick missing -> `401`.
- Target not in channel -> `441`.
- On success:
  - builds `KICK` message.
  - sends to target and rest of channel.
  - removes target from channel.

### INVITE
- Requires `nick channel` (`461`).
- Channel missing -> `403`.
- Requester not on channel -> `442`.
- Requester is not op -> `482`.
- Target nick missing -> `401`.
- Target already in channel -> `443`.
- On success:
  - adds target to invite list.
  - sends `INVITE` to target.
  - replies `341` to requester.

### TOPIC
- No params -> `461`.
- Channel missing -> `403`.
- Requester not in channel -> `442`.
- Query mode:
  - no topic -> `331`.
  - topic exists -> `332`.
- Set mode:
  - with `+t` and non-op requester -> `482`.
  - if allowed, sets topic and broadcasts `TOPIC`.

### MODE

#### Channel MODE
- Requires existing channel, requester member, and requester operator.
- Supports:
  - `+i/-i`: invite-only.
  - `+t/-t`: topic restricted to operators.
  - `+k/-k`: channel key.
  - `+l/-l`: user limit.
  - `+o/-o <nick>`: op/deop user.
- Errors:
  - missing argument -> `461`.
  - unknown mode -> `472`.
  - `+o/-o` target missing -> `401`.
  - target not in channel -> `441`.
- Every valid change is broadcast to the channel.

#### User MODE
- Only allowed for self-target; otherwise `502`.
- User modes are not implemented (fallback `472`).

## 8.5 Listings

### LIST
- Sends `321` start.
- No params: lists all channels (`322`).
- With params: lists only requested existing channels.
- Ends with `323`.

### NAMES
- No params: returns without action.
- Channel missing -> `403`.
- Channel exists -> sends `353` and `366`.

## 8.6 Output and disconnection

### QUIT
- Handler does not directly clean state.
- Calls `requestDisconnect(fd, reason)`.
- `requestDisconnect`:
  - marks `isQuitting=true` and sets `quitMsg`.
  - if output buffer empty -> disconnect now.
  - if pending output exists -> keeps `POLLOUT` active for flush.

### disconnectClient
On disconnect:
1. Determines `reason` (explicit quit or remote close).
2. `notifyClientExit` (QUIT broadcast to shared members, no duplicates).
3. Removes client from all joined channels.
4. Deletes channel if it becomes empty.
5. If client had a nick, `unbindNick`.
6. Removes fd from `_pfds`.
7. `close(fd)` and erase from `_clientsByFd`.

---

## 9) Full multi-user simulation (chronological)

Scenario: server + users `ana`, `bob`, `carla`.

1. Server starts on port 6667.
2. `ana` connects: `PASS/NICK/USER` -> receives welcome.
3. `bob` connects and registers.
4. `carla` connects and registers.
5. `ana` executes `JOIN #core`:
   - channel is created,
   - `ana` joins and becomes operator.
6. `bob` executes `JOIN #core`:
   - joins as member,
   - both receive join + names.
7. `ana` applies channel modes:
   - `MODE #core +i +t +k key +l 2`.
8. `carla` tries `JOIN #core` without invite/key:
   - fails due to restrictions (`473` or `475`).
9. `ana` invites:
   - `INVITE carla #core`.
10. `carla` retries with correct key:
   - joins if limit allows.
11. `bob` sends `PRIVMSG #core :hello`:
   - broadcast to other channel members.
12. `ana` sets topic:
   - `TOPIC #core :Sprint planning`.
13. `ana` grants op to `bob`:
   - `MODE #core +o bob`.
14. `bob` executes `KICK #core carla :off-topic`:
   - carla is removed from the channel.
15. `bob` executes `PART #core :bye`.
16. `ana` executes `QUIT :disconnecting`:
   - server flushes output and cleans all related state.
17. Process receives `SIGINT`:
   - `g_serverRunning=false`, loop exits,
   - destructor closes listening and remaining client sockets.

---

## 10) Timeout and inactivity

`checkTimeouts()` inspects every client, and if inactivity exceeds 600s:
- sends `ERROR :Closing link: Ping timeout (600 seconds)`.
- disconnects with standard cleanup.

---

## 11) Quick per-command validation (manual checklist)

- Registration:
  - wrong PASS -> 464.
  - NICK before PASS -> 451.
  - duplicate NICK -> 433.
- JOIN:
  - invalid channel -> 403.
  - `+k` without key -> 475.
  - `+i` without invite -> 473.
- MODE:
  - non-op setting channel MODE -> 482.
  - unknown mode -> 472.
- TOPIC:
  - non-op changing topic on `+t` channel -> 482.
- KICK:
  - target not in channel -> 441.
- PRIVMSG:
  - send to channel without membership -> 404.
- QUIT:
  - must notify shared members and clean nick/channel state.

---

## 12) Conclusion

The project implements an IRC server with clear separation between:
- **non-blocking network engine**,
- **protocol parsing**,
- **dispatcher-based routing**,
- **per-command handler logic**,
- **consistent client/channel state management**.

The disconnect flow (`QUIT` + flush + cleanup), channel modes (`i/t/k/o/l`), and membership management (JOIN/PART/KICK/INVITE) are coherently integrated into an event-driven `poll` model.
