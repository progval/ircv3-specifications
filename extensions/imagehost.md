---
title: The Imagehost ISUPPORT token
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
the unprefixed `IMAGEHOST` isupport token
Instead, implementations SHOULD use the `draft/IMAGEHOST`
isupport token to be interoperable with other software implementing
a compatible work-in-progress version.

The final version of the specification will use an unprefixed capability name.

# Motivation

This specification offers a way for network administrators to recommend
a hosting service for users to post images.

## Architecture

This specification introduces the `draft/IMAGEHOST` isupport token.

Its value MUST be a space-separated list of URIs, and it SHOULD use either the `http` or `https`.

Clients SHOULD ignore any value that isn't a URI (including the empty value),
or a URL that does not use either of these schemes.

When clients which to post an image using the network's recommended service,
they should send an HTTP POST to this URL.

If the response has the `201 Created` HTTP status code, they MAY read the `Location` header and use it as the URL of the uploaded image, which they can use as any URL.

