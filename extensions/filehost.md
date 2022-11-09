---
title: The Filehost ISUPPORT token
layout: spec
work-in-progress: true
copyrights:
  -
    name: "Val Lorentz"
    email: "progval+ircv3@progval.net"
    period: "2022"
---

## Notes for implementing work-in-progress version

This is a work-in-progress specification.

Software implementing this work-in-progress specification MUST NOT use
the unprefixed `FILEHOST` isupport token
Instead, implementations SHOULD use the `draft/FILEHOST`
isupport token to be interoperable with other software implementing
a compatible work-in-progress version.

The final version of the specification will use an unprefixed capability name.

# Motivation

This specification offers a way for network administrators to recommend a hosting service for users to upload files (such as images), so they can post them on IRC.

## Architecture

This specification introduces the `draft/FILEHOST` isupport token.

Its value MUST be a (possibly empty) space-separated list of URIs, and they SHOULD use either the `http` or `https` schemes. 

Clients SHOULD ignore any value that isn't a URI, or a URI that does not use either of these schemes.

When clients wish to post an image using the network's recommended service, they should send an HTTP POST to any of these URIs.

If the response has the `201 Created` HTTP status code, they MAY read the `Location` header and use it as the URI of the uploaded image, which they can use as any URI.

Clients SHOULD gracefully handle other common HTTP status codes that could occur, including, but not limited to:

* `401 Unauthorized`
* `403 Forbidden`
* `404 Not Found`
* `5xx` server errors

## Implementation Considerations

*This section is not normative*

Server implementations should take care to avoid abuse.

They may, for example, provide custom `draft/FILEHOST` URIs (with secret credentials) to clients after they authenticated with SASL.

## Client example view
The following example uses two example enpoints called `https://random.uri/` and `https://another.random.uri/`
```
:irc.server 005 YourNick draft/FILEHOST=https://random.uri/\x20https://another.random.uri/
```
