<!--
Copyright (C) 2000 - 2022 Daniel Stenberg, <daniel@haxx.se>, et al.

SPDX-License-Identifier: curl
-->

# WebSockets in curl

## API

The Websockets API is described in the individual man pages for the new API.

Websockets with libcurl can be done two ways.

1. Get the websockets frames from the server sent to a WS write callback. You
   can then respond with `curl_ws_send()` from within the callback or outside
   of it.

2. Set `CURLOPT_CONNECT_ONLY` to 2L (new for websockets), which makes libcurl
   do the `Upgrade:` dance in the `curl_easy_perform()` call and then you can
   use `curl_ws_recv()` and `curl_ws_send()` to receive and send websocket
   frames from and to the server.

The new options to `curl_easy_setopt()`:

 `CURLOPT_WS_OPTIONS` - to control specific behavior (no bits implemented yet)

The new function calls:

 `curl_ws_recv()` - receive a websockets frame

 `curl_ws_send()` - send a websockets frame

 `curl_ws_poll()` - NOT IMPLEMENTED YET

 `curl_ws_meta()` - NOT IMPLEMENTED YET

## Command line tool websockets

The plan is to make curl do websockets similar to telnet/nc. That part of the
work has not been started.

Ideas:

 - Read stdin and send off as messages. Consider newline as end of fragment.
   (default to text? offer option to set binary)
 - Respond to PINGs automatically
 - Issue PINGs at some default interval (option to switch off/change interval?)
 - Allow `-d` to specify (initial) data to send (should the format allow for
   multiple separate frames?)
 - Exit after N messages received, where N can be zero.

## Test suite

I looked for an existing small WebSockets server implementation with maximum
flexibility to dissect and cram into the test suite but I ended up deciding
that extending the existing test suite server sws to deal with WebSockets
might be the better way.

- This server is already integrated and working in the test suite

- We want maximum control and ability to generate broken protocol and negative
  tests as well. A dumber and simpler TCP server could then be easier to
  massage into this than a "proper" websockets server.


## Why not libwebsockets

[libwebsockets](https://libwebsockets.org/) is said to be a solid, fast and
efficient WebSockets library with a vast amount of users. My plan was
originally to build upon it to skip having to implement the lowlevel parts of
WebSockets myself.

Here are the reasons why I have decided to move forward with WebSockets in
curl **without using libwebsockets**:

- doxygen generated docs only makes them very hard to navigate. No tutorial,
  no clearly written explanatory pages for specific functions.

- seems (too) tightly integrated with a specific TLS library, while we want to
  support websockets with whatever TLS library libcurl was already made to
  work with.

- seems (too) tightly integrated with event libraries

- the references to threads and thread-pools in code and APIs indicate too
  much logic for our purposes

- "bloated" - it is a *huge* library that is actually more lines of code than
  libcurl itself

- websockets is a fairly simple protocol on the network/framing layer so
  making a homegrown handling of it should be fine