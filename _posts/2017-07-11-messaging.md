---
layout: page
title: "Messaging"
category: userguide/api
date: 2017-07-11 18:35:13
disqus: 1
---

Messaging allows users to send cross-webstrate messages between users, even when a user isn't
online. It's basically emails for Webstrates.

To send a message to a user (or several), call

```javascript
webstrate.message(message, recipients);
```

The `message` object can be any serializable JavaScript object (string, number, array, object...).
`recipients` can be either a `userId` (`username:provider`, e.g. `kbadk:github`), a current or
recently used `clientId`, or a list containing a combination of the former two.

```javascript
webstrate.message("Hello, world!", "kbadk:github")
```

and

```javascript
webstrate.message({ foo: ["quux", "bar"] }, ["B1bvxz0DNZ", "kbadk:github"])
```

are therefore both legal.

Only logged in users can send and receive messages.

Whenever a message is sent, it'll be delivered to all connected clients as well as saved on the
server. Listening for messages can be done using the `message` event on the global webstrate object.

```javascript
webstrate.on('message', function(message, senderId, messageId) {
  // message being the message object (e.g. "Hello, world!"), senderId being the sending
  // userId (e.g. "kbadk:github", never "B1bvxz0DNZ"), and messageId a random string
  // assigned to the message by the server (e.g. "rkwJJCP4Z").
});
```

This callback would also get triggered shortly after the `loaded` event has triggered in a document,
once for every message stored. As long as messages have not been deleted, these events will keep
getting triggered, until they expire. Messages automatically expire after 30 days.

Additionally, all messages are accessible in `webstrate.messages`.

To prevent the same messages from getting triggered on every load, they can be deleted, either by
deleting all messages:

```javascript
webstrate.deleteMessages();
```

Or by deleting single messages by messageId:

```javascript
webstrate.deleteMessage(messageId);
```

<div class="info box">
	<p><strong>A word on message persistence:</strong> Messages aren't automatically deleted from the
	server  after they have been received by a client, because we won't know if any of the receiving
	webstrates handles the messages appropriately.</p>
	<p>One can easily imagine a scenario where messages were intended for consumption in Webstrate A,
	but the recipient was only connected to Webstrate B at the time. As messages are addressed to
	users (not webstrates), Webstrate B would end up consuming the messages intended for Webstrate A
	as long as the user remained connected to the Webstrate B.</p>

	<p>Users are therefore required to manually delete messages to acknowledge that they have indeed
	handled the messages appropriately.</p>
</div>