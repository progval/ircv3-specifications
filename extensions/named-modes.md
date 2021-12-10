---
title: IRCv3 Named Modes Extension
layout: spec
copyrights:
  -
    name: "Attila Molnar"
    period: "2014"
    email: "attilamolnar@hush.com"
  -
    name: "Sadie Powell"
    period: "2017"
    email: "sadie@witchery.services"
  -
    name: "Val Lorentz"
    period: "2021"
    email: "progval+ircv3@progval.net"

---

## Intro

The name of this client capability MUST be named `named-modes`.

This capability enables receiving PROP messages instead of MODE messages as well as introducing the `RPL_CHMODELIST` and `RPL_UMODELIST` numerics to replace the user and channel mode parameters of the `RPL_MYINFO` numeric.

## New numerics on connection

These numerics MAY occur more than once. If the reply consists of multiple lines (due to IRC length limitations) all but the last numeric MUST have a parameter containing only an asterisk (*) preceding the mode list.

### RPL_CHMODELIST

    :<server name> XXX <nick> [*] {<type>:<modename>[=<letter>]}+

where `<type>` is one of the following that tells the client about the nature
of the mode:

* 1 - mode is a list mode, requires a parameter when setting and unsetting (group 1 in 005 CHANMODES).
* 2 - mode is a parameter mode, requires a parameter when setting and unsetting (group 2 in 005 CHANMODES).
* 3 - mode is a parameter mode, requires a parameter when setting, requires no parameter when unsetting (group 3 in 005 CHANMODES).
* 4 - mode is a flag, i.e. no parameter (group 4 in 005 CHANMODES).
* 5 - mode is a prefix mode, requires a parameter when setting and unsetting, target is always a user on the channel.

and `<letter>` is an equivalent mode name for the `MODE` command.

### RPL_UMODELIST

    :<server name> YYY <nick> [*] {<type>:<modename>[=<letter>]}+

where type is
* 3 - mode requires a parameter when setting, requires no parameter when unsetting.
* 4 - mode is a flag, it never has a parameter.

and `<letter>` is an equivalent mode name for the `MODE` command.

The list given by these two numerics are in two separate namespaces; it is
possible to have the same mode name for a user mode and a channel mode,
because the target of a mode change is always unambiguous.

When this capability is negotiated, the mode lists in RPL_MYINFO
(004) MUST be considered undefined, server implementations MAY
omit the last 3 parameters for that numeric entirely and clients MUST
handle this.
Servers MAY send no RPL_MYINFO numeric at all, clients
MUST handle this case as well.

Example 1: client connects and requests the named-modes capability

    Client: CAP LS
    Server: NICK tester
    Client: USER user 0 0 :gecos
    Server: :example.server CAP * LS :named-modes
    Client: CAP REQ named-modes
    Client: CAP END
    Server: :example.server CAP tester ACK :named-modes
    Server: :example.server 001 tester :Welcome to an IRC network, nick!user@example.com!
    Server: :example.server 002 tester :Your host is example.server, running version 0.0
    Server: :example.server 003 tester :This server was created ...
    Server: :example.server 004 tester :example.server
    Server: :example.server 005 tester :EXCEPTS=e NICKLEN=30 INVEX=I MAP MODES=4 NETWORK=Example
    Server: :example.server XXX tester :5:op=o 5:voice=v 4:private=p 4:secret=s 4:inviteonly=i 4:topiclock=t 4:noextmsg=n 4:moderated=m 3:limit=l 1:ban=b 2:key=k
    Server: :example.server YYY tester :4:oper=o 4:invisible=i 1:snomask=s 4:wallops=w

## Listing modes on a channel

PROP <channel>

Reply syntax:

    RPL_PROPLIST <nick> <channel> :[<modename>[=<param>]] [<modename 2>[=<param 2>]] ... [<modename n>[=<param n>]]]
    RPL_ENDOFPROPLIST <nick> <channel> <modename> :End of list

Servers MAY send multiple `RPL_PROPLIST` replies before `RPL_ENDOFPROPLIST`.


    PROP #example
    :test.server 961 modernclient #example :topiclock noextmsg limit=5
    :test.server 960 modernclient #example :End of mode list

## Getting the list of a listmode on a channel

Query syntax:

    PROP <channel> <modename>

Reply syntax:

    RPL_LISTPROPLIST <nick> <channel> <modename> :<mask> [<setter> <settime>]
    RPL_ENDOFLISTPROPLIST <nick> <channel> <modename> :End of list

Servers MAY send multiple `RPL_PROPLIST` replies before `RPL_ENDOFPROPLIST`.

The `settime` argument, if present, MUST be a UNIX timestamp of the time this mode
was placed.
The `setter` argument, if present, SHOULD be the nickname, `nick!user@host` mask,
or server name of the entity who placed this mode.

Example without the optional arguments:

    Client: PROP #chat ban
    Server: :example.server BBB tester #chat ban :*!*@example.org
    Server: :example.server BBB tester #chat ban :another!banned@user.example.com
    Server: :example.server AAA tester #chat ban :End of list

Example with the optional arguments:

    Client: PROP #opers ban
    Server: :example.server BBB tester #chat ban :*!*example.org mike!mike@localhost 567890123
    Server: :example.server BBB tester #chat ban :*!*@192.0.2.69 ChanServ!ChanServ@services.example.com 123123123
    Server: :example.server BBB tester #chat ban :*!*@192.0.2.70 ChanServ!ChanServ@services.example.com 123123123
    Server: :example.server AAA tester #chat ban :End of list


## Changing modes

A "mode change" means a `PROP` command that has a target and one or more
mode names and their associated parameters (if required by the mode).

An "item in the mode change" is defined as one mode and its parameter (if any)
in a mode change. A mode change MUST have at least one item in it.

A mode change from the client to the server is always a request; it being sent
does not guarantee the server will honor it, even if the client (thinks) it has
all the necessary priviliges, etc.

A mode change from the server to the client is a notification about a change
which had already happened on the server by the time the client receives the
command.

`PROP` command

Syntax:

    PROP <target> {<+|-><modename>[=<parameter>]}+

Add (+) or remove (-) the mode called `<modename>`, using `<parameter>` as
the parameter, if required for the mode.

Client implementations MUST NOT send parameters for modes that do not require
one.
Server implementations MUST remove the meaningless parameter (and the
accompanying `=` sign) before propagating the mode change to other clients.

A mode change is a "no-op mode change" if after being processed the server,
the server ends up not changing anything.

If the mode change is not a no-op mode change it MUST be echoed back to the
client doing the change after being processed.
Servers SHOULD normally send the change to all other clients if the target
is a channel, in a similiar way to how they behave with a RFC 1459 `MODE`
command.

If a server implementation ecounters an item in the mode change with a
parameter which is invalid according to it, it is allowed to change (fix)
the parameter instead of rejecting that item in the mode change.
Clients MUST be prepared to handle a mode change echoed back to them
which has an item whose parameter is slightly or entirely different than
what was requested by the client.

A mode change from the client to the server SHOULD have at most MAXMODES
(from RPL_ISUPPORT) items. The surplus items SHOULD be treated the same way
by the server as it treats surplus modes in a RFC 1459 `MODE` command.
Usually this means that the surplus items are ignored silently and the `PROP`
and `MODE` messages generated by this mode change won't contain the surplus
items.
This behavior not required, meaning servers can choose to process
more items if they wish, for example, if the mode change comes from a
priviliged client the server is free to be more lax and process all items.

A mode change from the server to the client MAY contain an arbitrary
number of items. However it still MUST honor other limitations imposed
by the protocol such as the line length limit.
Note that this is different than how the traditional `MODE` command behaves,
which usually only contains up to MAXMODES changed modes.

## Translation between `PROP` and `MODE`

When a client send a `PROP` message that should be relayed to other clients,
servers MAY translate it to a `MODE` command using an equivalent
mode letter (as defined in `RPL_CHMODELIST`) to clients that did not
negotiate this capability, but MUST NOT send them a `PROP`.

When a client sends either `PROP` or `MODE`, servers SHOULD send them as
a `PROP` to clients who negotiated this capability.
Clients MUST accept `MODE` messages.

## Examples

*This section is not normative*

Changing channel modes

    Client: PROP #egypt :+key=pyramids -topiclock +ban=*!*@example.com +ban=example!*@*

If the mode change is successful the following (or an equivalent) is sent to
all clients supporting this capability:

    Server: :nick!user@host PROP #egypt :key=pyramids -topiclock ban=*!*@example.com ban=example!*@* 

Clients not supporting this capability receive the following (or an
equivalent) mode change:

    Server: :nick!user@host MODE #egypt +kbb-t pyramids *!*@example.com example!*@*


## Mode names

### Rules for choosing mode names

`<modename>` in the grammars above is defined as follows:

    <modename>        ::= [ <prefix> ] [ <vendor> '/' ] <key_name>
    <unqual_modename> ::= <non-empty sequence of ascii letters, digits, hyphens ('-')>
    <vendor>          ::= <host>

They follow [the same rules as message tag names](../extensions/message-tags.html#rules-for-naming-message-tags).
The following sections define names for existing standard modes.

### RFC 1459 channel modes

Letter | Name
------ | ------------
 `o`   | `op`
 `v`   | `voice`
 `b`   | `ban`
 `i`   | `inviteonly`
 `l`   | `limit`
 `m`   | `moderated`
 `n`   | `noextmsg`
 `k`   | `key`
 `p`   | `private`
 `s`   | `secret`
---------------------

### RFC 1459 user modes

Letter | Name
------ | ------------
 `i`   | `invisible`
 `o`   | `oper`
 `w`   | `wallops`
 `s`   | `snomask`
---------------------

### Numerics

New numerics that this spec defines are:

| No. | Label                   |
| --- | ----------------------- |
| 960 | `RPL_ENDOFPROPLIST`     |
| 961 | `RPL_PROPLIST`          |
| AAA | `RPL_ENDOFLISTPROPLIST` |
| BBB | `RPL_LISTPROPLIST`      |
| XXX | `RPL_CHMODELIST`        |
| XXX | `RPL_UMODELIST`         |
