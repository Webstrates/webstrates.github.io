---
layout: page
title: "Client Management"
category: userguide/api
date: 2018-10-03 16:00:38
---

* TOC
{:toc}

Building collaborative applications is usually easier if you, as a developer, have access to information about the other collaborators/clients as well as information about your own clients. Webstrates facilitates this through its client list and user object functionality.

## Client list

A full list of clients connected to a webstrate can be accessed through `webstrate.clients`.
The list contains the Client IDs of all clients currently connected to the webstrate.
A call to `webstrate.clients` from a webstrate with 4 clients connected could look something like this:

```javascript
["ryr0xnt57", "BJNwZnFqQ", "H14DZ3tqm", "ryrPW3YcX"]
```

The client's own Client ID can be found in `webstrate.clientId`.

Other than to simply identify clients, these Client IDs are also used to [send signal between clients](/userguide/api/signaling.html) as well as [sending messages](/userguide/api/messaging.html).

Client IDs are randomly generated and not persisted between sessions, i.e. leaving and rejoining a webstrate will give the client a new Client ID.

## Client join and part events

To monitor the comings and goings of clients, the user may subscribe to the `clientJoin` and `clientPart` events.
To see a full list of events and how to subscribe to them, see the [Events documentation page](/userguide/api/events.html#full-list-of-on-events).

```javascript
webstrate.on("clientJoin", function(clientId) {
  // User with clientId has just joined the webstrate.
});
```

## User object

When a user is logged in with an OAuth provider, the `webstrate.user` object will be defined, containing a bunch of information about the current user:

- `allClients` -- an object containing all user's currently connected clients to any webstrate. This object also includes the webstrateId, the user's IP address, as well as the user's [user agent string](https://developer.mozilla.org/en-US/docs/Glossary/User_agent).
- `clients` -- a simplified list of `allClients`, only containing Client IDs (same format as `webstrate.clients`).
- `cookies` -- all the user's [cookies](/userguide/api/cookies.html).
- `displayName` -- the user's display name as defined by the OAuth provider.
- `on` and `off` -- functions for attaching event listeners for things like [user object signaling](/userguide/api/signaling.html#signaling-on-user-object).
- `permissions` -- the user's permissions in the current document.
- `provider` -- the identifier for the OAuth provider.
- `userId` -- the user's `username` and `provider` joined together by a `:`. Note that a user's User ID is different from their Client ID.
- `userUrl` -- a URL to the user's homepage as defined by the OAuth provider.
- `username` -- the user's username as defined by the OAuth provider.
- `signal` -- a function used to send signals on the user object.