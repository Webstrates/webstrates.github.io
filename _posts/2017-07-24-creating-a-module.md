---
layout: page
title: "Creating a module"
category: developerguide
date: 2017-07-24 16:26:15
disqus: 1
---

* TOC
{:toc}

It may be useful to expand the functionality of Webstrates with additional client modules; modules that can provide functionality to all webstrates.

Creating a new module can be useful both for providing functionality that is used in many webstrates, lest having to reimplement the same thing over and over.
Modules can also provide new functionality that can't be implemented solely in client code, either because the module will depend on internal events or it needs to use the websocket, possibly because it needs to handle new kinds of websocket data.
In this case, there is currently no documentation on how to expand the server side code, only the client code.

We'll now go over two example modules that will demonstrate how to create a module for Webstrates.

## Party module

They say that "Three's a party", so let's implement a very basic module that creates a global `party` event which gets triggered when a third user joins a webstrate.

Modules are located in the `./client/webstrates/` folder in the Webstrates installation directory, so let's start out by creating a new JavaScript file at `./client/webstrates/party.js`.

After creating the module file, we need to tell Webstrates to use the module. This is done by adding `'party'` to the end of [the `modules` list in the client configuration `./client/config.js`](/userguide/client-config.html#modules).

Our module will need to include two of [the built-in modules](/developerguide/introduction.html#list-of-modules), namely `coreEvents` and `globalObject`.
We want to listen for the `clientJoin` event, so we need `coreEvents`. We also need to create a new event on the global object (i.e. `window.webstrate`) for which we need the `globalObject` module.

```javascript
'use strict';
const coreEvents = require('./coreEvents');
const globalObject = require('./globalObject');
```

Now, all we need to do is to create the `party` event.

```javascript
globalObject.createEvent('party');
```

And trigger the event when the number of clients reaches 3:

```javascript
coreEvents.addEventListener('clientJoin', (clientId) => {
  if (webstrate.clients.length === 3) {
    globalObject.triggerEvent('party', clientId);
  }
});
```

And that's it. After rebuilding the client application code (by calling `npm run build`), the module
will be active and may be used in any webstrate.

Adding the following script to any webstrate will thus cause "Three's a party" -- followed by the joining client's `clientId` -- to get printed to the console whenever a third user joins the webstrate.

```javascript
webstrate.on('loaded', () => {
  webstrate.on('party', (clientId) => {
    console.log('Three\'s a party!', clientId);
  });
});
```

## Smash module

Let's make it possible to draw attention to a specific `div` element in the DOM on other clients.
We'll do this by allowing users to run `DOMNode.webstrate.smash()` on a `div` element, and let other clients listen for these smashes with `DOMNode.webstrate.on('smash', fn)`.

"Smashing" an element seems very similar to signaling, so we will in fact be building this module on top of signaling.

To create the event, we need to wait for all webstrate objects to have been added, so we again need `coreEvents`.
Since we're building this on top of signaling, we also need the `signaling` module.

```javascript
'use strict';
const coreEvents = require('./coreEvents');
const signaling = require('./signaling');
```

Before we can create the `smash` event and `smash()` function, we need to wait for the webstrate
objects to get added to the DOM elements.

Once the document has been created initially, and webstrate objects have been added, the `webstrateObjectsAdded` event gets triggered with a list of all DOM nodes (and event objects) in the document with webstrate objects on.
When later on, a new element gets added to the document, the `webstrateObjectAdded` event gets triggered with the node and event object as well.
Note that the event names differ slightly; the former is plural (Objects) and the latter is singular (Object).

If we listen for these two events, we can thus ensure that our code will get called whenever a webstrate object gets added.

```javascript
coreEvents.addEventListener('webstrateObjectsAdded', (nodes) => {
  nodes.forEach((eventObject, node) => makeSmashable(node, eventObject));
}, coreEvents.PRIORITY.IMMEDIATE);

coreEvents.addEventListener('webstrateObjectAdded', (node, eventObject) => {
  makeSmashable(node, eventObject);
}, coreEvents.PRIORITY.IMMEDIATE);
```

<div class="info box">
  <h3>What does <code>coreEvents.PRIORITY.IMMEDIATE</code> mean?</h3>
  <p>
    The second argument to <code>coreEvents.addEventListener()</code> is a priority, which determines the order in which the event handlers are triggered.
    This can be useful if, for instance, one event handler has to have finished its job before another event handler can perform its own job.
  </p>
  <p>
    There's a total of 5 priorities, namely <code>IMMEDIATE</code>, <code>HIGH</code>, <code>MEDIUM</code>, <code>LOW</code>, and <code>LAST</code>.
  </p>
  <p>
    Any event handler with <code>IMMEDIATE</code> priority gets executed straight away in the same execution.
    The use of the <code>IMMEDIATE</code> priority should be limited as much as possible, as doing too much work in the same task can cause the browser to become unresponsive.
    However, in this case, we need to run our code immediately, as otherwise clients may attempt to add an event listener for the <code>smash</code> event before it has been created.
  </p>
  <p>
    Event handlers attached with the other 4 priorities are run through <a href="https://developer.mozilla.org/en/docs/Web/API/Window/setImmediate"><code>setImmediate()</code></a> (or <a href="https://developer.mozilla.org/en/docs/Web/API/Window/setTimeout"><code>setTimeout()</code></a> if <code>setImmediate</code> isn't supported by the browser), thus getting executed in their own task.
    Tasks with <code>HIGH</code> priority are the first to be executed (after <code>IMMEDIATE</code>), followed by tasks with <code>MEDIUM</code>, <code>LOW</code>, and finally <code>LAST</code> priorities, respectively. Event handlers added without priority default to <code>LOW</code> priority.
  </p>
</div>

In the above, we make sure that `makeSmashable(node, eventObject)` (a function we'll implement now) gets run for all DOM nodes in the document.

```javascript
const widEventMap = new Map();

function makeSmashable(node, eventObject) {
  if (node instanceof HTMLDivElement) {
    eventObject.createEvent('smash');

    Object.defineProperty(node.webstrate, 'smash', {
      value: () => {
        node.webstrate.signal('__smash');
      }
    });

    widEventMap.set(node.webstrate.id, eventObject);
    signaling.subscribe(node.webstrate.id);
  }
}
```

In the above, we check if our node is a `div` elemnt, and if so we create the `smash` event and define the `smash()` function.
The `smash` function simply sends a signal with the contents `__smash` on the node.
Lastly, we subscribe to signals on the node and create and populate a map (`widEventMap`) from the elements' `__wids` to their respective event objects, which we will be using in the next and final part of our module.

<div class="info box">
  <p>
    <strong>What are those <code>__wid</code>s?</strong> To keep better track of DOM, Webstrates
    adds unique IDs to each DOM elements. These are usually not visible in the DOM, even though they
    exist in the JsonML representation. While it should rarely be necessary for users to access
    the <code>__wid</code> of an element, it is available as <code>id</code> on all non-transient
    DOM nodes' webstrate objects (i.e. <code>DOMNode.webstrate.id</code>).
  </p>
  <p>
    If you want to know more about JsonML, check out
    <a href="/developerguide/how-it-works.html#jsonml">How it works</a> in the Developer guide.
  </p>
</div>

In order to avoid signal event handlers getting triggered by our `smash` event, we install an interceptor on all signals.
The interceptor works by receiving all signals we're subscribed to, and then returning `true` if the signal needs to be interceptor (i.e. not propagated further) or not.
If the interceptor returns a falsy value, the usual signaling related events (both system and userland) get triggered.

```javascript
signaling.addInterceptor(payload => {
  if (payload.m !== '__smash') return false;

  const eventObject = widEventMap.get(payload.id);
  if (!eventObject) return false;

  eventObject.triggerEvent('smash', payload.s);
  return true;
});
```

In our interceptor, we access the raw payload object. This object contains a few pieces of information, such as the message itself (`payload.m`), the sender's clientId `payload.s`, and the `__wid` of the element the signal was sent to (`payload.id`).
We first make sure that the signal received is in fact a smash, or otherwise return.
Afterwards, the map we created before comes into play. We look up the `__wid` in the map to find the event object associated with the node to whom the `__wid` belongs.
If we can't find the event object, the shouldn't intercept, either.
On the other hand, if we can find the event object, we trigger the event (with the sender's clientId as argument) and return `true` to intercept the signal.

After it has been installed and enabled (as with the Party module), it can be used in a webstrate as below.

```html
<html>
<head>
  <script>
  webstrate.on('loaded', () => {
    Array.from(document.querySelectorAll('div'), node => {
      node.webstrate.on('smash', senderId => {
        console.log('smash on', node.innerText);
      });

      node.addEventListener('click', () => {
        node.webstrate.smash();
      });
    });
  });
  </script>
</head>
<body>
  <div>A</div>
  <div>B</div>
  <div>C</div>
</body>
</html>
```

In the above webstrate, clicking on one of the `div`s will result in `smash on`, followed by either `A`, `B`, or `C` to be printed in the console of all connected clients.