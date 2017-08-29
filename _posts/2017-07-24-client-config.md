---
layout: page
title: "Client Configuration"
category: userguide
date: 2017-07-24 16:11:50
disqus: 1
---

* TOC
{:toc}

The client configuration can be found in `./client/config.js` and is (contrary to the server
configuration (`./config.json`) a JavaScript file rather than a JSON file. The file exports an
object with properties (using the `module.exports = { ...};` syntax).

After changing the configuration, remember to re-run `npm run build`. Depending on [server
configuration caching settings](/userguide/server-config.html#basic-settings), it may also be
necessary for clients to clear caches or perform a hard refresh.

<div class="info box">
	<h3>What does hard refresh mean?</h3>
	<p>Many web applications (including Webstrates) are configured to tell the browser to cache
	certain resources, (i.e. save the resources between page reloads) to save bandwidth and speed-up
	page load times. To circumvent this, all modern browsers implement a "hard refresh" feature that
	forces the browser to redownload caches resources.</p>
	<ul>
		<li>
			<strong>Apple Safari</strong>:
			Hold down the Command key <kbd>&#8984;</kbd> and Shift key <kbd>&#8679;</kbd> while pressing
			the <em>Reload this page</em> button.
		</li>
		<li>
			<strong>Microsoft Edge</strong>:
			Hold down the Control key <kbd>CTRL</kbd> while pressing <kbd>F5</kbd>.
		</li>
		<li>
			<strong>Google Chrome and Mozilla Firefox</strong>:
			On macOS: Same as Apple Safari. On Linux and Windows: Same as Microsoft Edge.
		</li>
	</ul>
</div>

## Modules

Webstrates is built around events and modules subscribing and reacting to said events. The `modules`
propery in the configuration contains a list of modules to import from the `./client/webstrates/`
folder.

```javascript
modules: [
  'globalObject',
  'loadedEvent',
  'userObject',
  'cookies',
  'nodeObjects',
  'domEvents',
  'transclusionEvent',
  'connectionEvents',
  'permissions',
  'tagging',
  'clientManager',
  'signaling',
  'signalStream',
  'assets',
  'messages',
  'keepAlive'
],
```

The client code also consists of certain "core" modules, containing core functionality (also located
in `./client/webstrates`), but these are loaded automatically and should not be included in the
list.

The order of the items should not be changed as some modules depend on each other, and thus must be
loaded in a specific order. User-created modules should be added to the bottom of the list.

<div class="info box">
	<p>See <a href="/developerguide/creating-a-module.html">Creating a module</a> in the Developer
	guide to learn how to create custom modules.</p>
</div>

## Transient elements and attributes

By default, all `<transient`> elements are (rather appropriately)
[transient](/userguide/api/transient#transient-elements). However, the client configuration
allows the user to define a predicate function defining whether a given
[DOM Node](https://developer.mozilla.org/en/docs/Web/API/Node) should be transient.

```javascript
isTransientElement: (DOMNode) => DOMNode.matches('transient')
```

Likewise for attributes, any attribute's name which starts with `transient-` is considered
transient. The user may also define a custom predicate function here to customize this behavior.

```javascript
isTransientAttribute: (DOMNode, attributeName) => attributeName.startsWith('transient-')
```

To disable transient tags and attributes altogether, one could replace the default implementations
with dummy predicate functions that always return `false`.

## Websocket configuration

`reuseWebsocket` decides whether Webstrates should use the experimental websocket usage feature.
When this is enabled, transcluded webstrates will attempt to reuse the parent frame's websocket.
This is very experimental and should probably be disabled.

```javascript
reuseWebsocket: true
```

`keepAliveInterval` defines the interval in seconds between keep alive messages are sent to the
server. Some systems may close websockets that have been inactive for a minute or longer.

Setting this to a falsy value disables keep alive messages. Alternatively, `keepAlive` can be
removed from the `modules` array should the user want to disable keep alive messages (not
recommended).

```javascript
keepAliveInterval: 55
```

## WebRTC configuration

[Signal streaming](/userguide/api/signaling.html#signal-streaming) uses WebRTC. To establish WebRTC
connections behind a NAT, STUN servers are used. The default configuration includes Mozilla and
Google's free STUN servers.

```javascript
peerConnectionConfig: {
  'iceServers': [
    { url: 'stun:stun.services.mozilla.com' },
    { url: 'stun:stun.l.google.com:19302' }
  ]
}
```

In some cases (e.g. when behind corporate firewalls), a STUN server isn't sufficient, in which case
users may have to add their own TURN server to the list.