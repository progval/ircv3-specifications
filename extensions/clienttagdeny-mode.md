---
title: "Clienttagdeny Mode"
layout: spec
work-in-progress: true
copyrights:
  -
    name: "Val Lorentz"
    period: "2022"
    email: "progval+ircv3@progval.net"
extends:
  - named-modes
  - message-tags
---


## Notes for implementing work-in-progress version

This is a work-in-progress specification.

Software implementing this work-in-progress specification MUST NOT use the
unprefixed `clienttagdeny` mode. Instead, implementations SHOULD
use the `draft/clienttagdeny` mode to be interoperable with other
software implementing a compatible work-in-progress version.

The final version of the specification will use an unprefixed mode name.


## Introduction

This mode is intended to allow channel operators to configure what [client tags][message-tags]
are allowed in their channel.



## Architecture

This specification extends the [message-tags][message-tags] specification,
by adding a new mode named `draft/clienttagdeny`.
To avoid conflicts with existing modes, it is only available to software
implementing the [named-modes][named-modes] specification.

### Mode

The `draft/clienttagdeny` mode requires a parameter when setting and no parameter
when unsetting (group 3 in 005 CHANMODES and the named-modes specification).


### Semantics

The parameter MUST follow the syntax of the [CLIENTTAGDENY ISUPPORT token][message-tags#rpl_isupport-tokens],
and is interpreted in the same way.


### Integration with the ISUPPORT token

When a network advertizes a `CLIENTTAGDENY` and an a `draft/clienttagdeny` is set,
servers SHOULD only allowed tags allowed by both policies.

## Examples

These examples assume the `echo-message` capability is negotiated as well,
to show how servers rewrites the client's messages

    C: @+example.org/tag1=value1,+example.org/tag2=value2 PRIVMSG #channel :by default, all client tags are allowed
    S: @+example.org/tag1=value1,+example.org/tag2=value2 :nick!user@host PRIVMSG #channel :by default, all client tags are allowed

    C: PROP #example draft/clienttagdeny=example.org/tag1
    S: :nick!user@host PROP #example :draft/clienttagdeny=example.org/tag1
    C: @+example.org/tag1=value1,+example.org/tag2=value2 PRIVMSG #channel :now, all tags but example.org/tag1=value1 are allowed
    S: @+example.org/tag2=value2 :nick!user@host PRIVMSG #channel :now, all tags but example.org/tag1=value1 are allowed

    C: PROP #example draft/clienttagdeny=*,-example.org/tag1
    S: :nick!user@host PROP #example :draft/clienttagdeny=*,example.org/tag1
    C: @+example.org/tag1=value1,+example.org/tag2=value2 PRIVMSG #channel :now, only example.org/tag1=value1 is allowed
    S: @+example.org/tag1=value1 :nick!user@host PRIVMSG #channel :now, only example.org/tag1=value1 is allowed

    C: PROP #example -draft/clienttagdeny
    S: :nick!user@host PROP #example :-draft/clienttagdeny
    C: @+example.org/tag1=value1,+example.org/tag2=value2 PRIVMSG #channel :all tags are allowed again
    S: @+example.org/tag1=value1,+example.org/tag2=value2 :nick!user@host PRIVMSG #channel :all tags are allowed again

[message-tags]: ../extensions/message-tags.html
[message-tags#rpl_isupport-tokens]: ../extensions/message-tags.html#rpl_isupport-tokens
[named-modes]: ../extensions/named-modes.html
