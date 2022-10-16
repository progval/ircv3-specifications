---
title: "Nick Recovery Extension"
layout: spec
work-in-progress: true
copyrights:
  -
    name: "Val Lorentz"
    period: "2022"
    email: "progval+ircv3@progval.net"
---

## Notes for implementing work-in-progress version

This is a work-in-progress specification.

Software implementing this work-in-progress specification MUST NOT use the
unprefixed `nick-recovery` capability name. Instead, implementations SHOULD
use the `draft/nick-recovery` capability name to be interoperable with other
software implementing a compatible work-in-progress version.

The final version of the specification will use an unprefixed capability name.


## Introduction

Most IRC servers disallow use of the same nick from mor than one connection at the same time.
This prevents users from using their nick when they connect and log-in when their nick is already in use, even by unauthenticated users or their previous, broken, connection.

Historically, this problem is solved by providing NickServ commands (`GHOST`, `REGAIN`, `RECOVER`, `RELEASE`, ...) to close or change the nick of existing connections using a nick owned by the issuer.
This specification introduces first-class IRC commands, which clients can programmatically discover.


## Architecture

### Dependencies

This specification uses [standard replies][] framework.

### Capability

This specification adds the `draft/nick-recovery` capability, whose presence signifies that the server accepts the `RECOVER` command.

Clients MUST ignore this capability's value.




### Command

    RECOVER <nick> [NOCHANGE]

The `RECOVER` command requests that the server changes the nick of (or closes) any connection currently using the given nick.

If the command succeeds, then the client's nick MUST change, unless the client provided the `NOCHANGE` flag.

Servers SHOULD respond with either `RECOVER SUCCESS` and `NICK`, or a standard reply.
The parameter of the `NICK` sent as a reply SHOULD be the one requested by the client.

### Responses

    RECOVER SUCCESS <nick> <message>

Indicates a `RECOVER` command sent by the client succeeded.
`<nick>` MUST be the same nick as the one requested by the client, but MAY be casefolded.

`RECOVER SUCCESS` MUST be followed by a `NICK` command.

    FAIL RECOVER NOT_AUTHENTICATED <nick> <message>

Sent by the server if the client is not authenticated.

    FAIL RECOVER NOT_AUTHORISED <nick> <message>

Sent by the server if the client is authenticated to an account that does not own the requested nick.

    FAIL RECOVER NOT_REGISTERED <nick> <message>

Sent by the server if the requested nick is not registered (ie. no account may disconnect it).

    FAIL RECOVER UNAUTHENTICATED_TARGET <nick> <message>

Sent by the server if the requested nick is not authenticated not the account that owns the nick -- if the server has such a requirement.

    FAIL RECOVER TEMPORARILY_UNAVAILABLE <nick> <message>

Sent by the server if the `RECOVER` commands are temporarily unavailable and/or temporarily cannot be used on the given nick.


## Implementation Considerations

Servers which allow multiple connections to the same nick (including bouncers) typically do not need to implement this specification, except to forward commands to upstream servers.

When sent during connection registration, servers MAY delay the effect of `RECOVER` (closing the other connection) until `CAP END` is sent, but not the response to the command, so this is transparent to the client.


[standard replies]: ../extensions/standard-replies.html
