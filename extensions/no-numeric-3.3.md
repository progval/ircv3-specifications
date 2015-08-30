---
title: IRCv3.3 `no-numeric` Extension
layout: spec
copyrights:
  -
    name: "Valentin Lorentz"
    period: "2015"
    email: "progval+ircv3@progval.net"
---
## The no-numeric Capability

When `no-numeric` is successfully negociated, the server MUST NOT send
numeric commands to the client. Instead it can use one of the alphabetical
replacements defined here, in other extensions' specification, or
vendor-defined command names.

If `no-numeric` has not been negociated, the server MUST still use
RFC 2812 numeric codes instead of their alphabetical translation and
MAY use translation for other (ie. non-standard) numeric commands.

Of course, clients still MUST NOT send numeric commands to servers,
whether or not this capability has been negociated.

This specification does not imply any other change of behavior
from servers or clients.
The table of translation of non-standard numerics does not make
them standard, it only exists as a hint for vendor-defined command names.

### Vendor-specific command names

TODO

### Translation of existing numeric commands

#### RFC 2812 and IRCv3

Numerics defined in [section 5 of RFC 2812][rfc2812-sec5]
or in other parts of the IRCv3 specification are translated to their
label.
Examples: `001` becomes `RPL_WELCOME` and `760` becomes `RPL_WHOISKEYVALUE`.

[rfc2812-sec5]: https://tools.ietf.org/html/rfc2812#section-5

#### Other codes

| No. | Translation             | Meaning
| --- | ----------------------- | --------
| 329 | ??                      | Sent when joining a channel, contains the creation timestamp.
| 330 | ??                      | Reply to whois, containing account name. XXX: deprecate this? Or use `ACCOUNT` from [`account-notify`][account-notify] ?
| 354 | ??                      | Used by WHOX
| 435 | ??                      | Can't change nick because of channel ban
| 438 | ??                      | Can't change nick, custom server message
| 504 | `ERR_NOSUCHNICK`        | Looks equivalent to 401
| 515 | ??                      | Cannot join, not authenticated

[account-notify]: http://ircv3.net/specs/extensions/account-notify-3.1.html
