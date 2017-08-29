---
layout: page
title: "Signaling"
category: userguide/api
date: 2017-07-07 00:12:57
disqus: 1
---

* TOC
{:toc}

Subscribe to signals on DOM elements and receive messages from other clients sending signals on the DOM element. Useful especially for huge amounts of data that will be expensive to continuously publish through the DOM (e.g. sensor data).

Listen for messages on `elementNode`:

```javascript
elementNode.webstrate.on("signal", function(message, senderId, node) {
  // Received message from senderId on node.
});
```

`node` will always equal `elementNode`, but may be useful if the node is no longer in scope.

Send messages on `elementNode`:

```javascript
elementNode.webstrate.signal(message, [recipients]);
```

An optional array of Client IDs (`recipients`) can be passed in. If recipients is not defined, all subscribers will recieve the message, otherwise only the clients in `recipients` will. Recipients are never aware of who else has received the signal.

Instead of listening for specific signals on DOM nodes, it is also possible to listen for all events using the webstrate instance.

```javascript
webstrate.on("signal", function(message, senderId, node) {
  if (node) {
    // Signal sent on node.
  } else {
    // Signal sent on webstrate instance.
  }
});
```

If the signal was sent on the webstrate instance (`webstrate.signal(message, [recipients])`), `node` will be undefined, otherwise `node` will be the DOM element the signal was sent on.

Listening on the webstrate instance does not circumvent the recipients mechanism -- subscribers will still not recieve signals specifically addressed to other clients.

### Signaling on user object

In addition to signaling on DOM elements, it is also possible to signal on the user object, allowing users to send signals/messages to any of the user's other connected clients across different webstrates:

```javascript
webstrate.user.on("signal", function(message, senderId) {
  // A message was received from client with senderId.
));
```

When sending a signal, no `recipients` are specified, because all user object signals automatically are sent to all connected clients, regardless of which webstrate the client is connected to.

```javascript
webstrate.user.signal(message);
```
## Signal streaming

Webstrates supports WebRTC-based signaling streaming, allowing users to stream to each other through nodes directly (peer-to-peer).

A user may start listening for users interested in receiving a stream on a node using:

```javascript
elementNode.webstrate.signalStream(function signalStream(clientId, accept) {
  // User with clientId is listening for streams on elementNode.
});
```

If the user agrees to let clientId receive a stream, the user calls the accept callback provided as the second argument with the stream as well meta data (any serializable object), and an optional callback that'll be called once the connection has been established. The accept function returns a connection object.

```javascript
elementNode.webstrate.signalStream(function signalStream(clientId, accept) {
  var conn = accept(stream, meta, function() {
    // Connection has been established.
  });
});
```

Listening for streams on a node is done using:

```javascript
elementNode.webstrate.on("signalStream", function(onSignalStream(clientId, meta, accept) {
  // User with clientId is requesting to send a stream.
});
```

If the user wants to accept the stream, the connect function is called with a callback that will be triggered once the stream is ready:

```javascript
elementNode.webstrate.on("signalStream", function(onSignalStream(clientId, meta, accept) {
  var conn = accept(function(stream) {
    // Stream received.
  });
});
```

If either user does not want to initiate the connection, they simply abstrain from calling the `accept` callback.

### Terminating streams

Either user can at any time terminate the stream by calling `conn.close()`. If a connection is terminated, the other user will get notified if they have added an `onclose` callback to the connection object:

```javascript
conn.onclose(function() {
  // Connection was terminated.
});
```

To stop listening for clients listening for streams, the streaming client may do:

```javascript
elementNode.webstrate.signalStream(function signalStream(clientId, accept) {
  var conn = accept(stream, meta, function() {
    // No longer accepting requests to receive the stream.
    elementNode.webstrate.stopStreamSignal(signalStream);
  });
});
```

The already established connections will not be terminated, but no new requests will be received. Note that `stopStreamSignal` has to be called _after_ the connection to the client has been established, otherwiser the connection will never finish getting established.

Clients listening for streams do not have to stop listening for new requests. Once one signal stream has been established, the client will automatically stop listening for additional streams. If the client wants to receive multiple streams, it can simply start listening for streams on the node again.

### Stream types

The signal streaming can carry any [`MediaStream`](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream) object, for instance a video and audio feed:

```javascript
elementNode.webstrate.signalStream(function signalStream(clientId, accept) {
  // Get audio and video feed.
  navigator.mediaDevices.getUserMedia({ audio: true, video: true }).then(function(stream) {
    // And send it to the requesting client.
    var meta = { title: "My Video Stream" };
    var conn = accept(stream, meta, ...);
  });
});
```

The stream received (in this example) can then be added to a `<video>` element in the DOM.

```javascript
var videoElement = document.createElement("video");
document.body.appendChild(videoElement);
elementNode.webstrate.on("signalStream", function onSignalStream(clientId, meta, accept) {
  var conn = accept(function(stream) {
    // Add stream to video element and play it.
    videoElement.srcObject = stream;
    videoElement.play();
  });
});
```