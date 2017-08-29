---
layout: page
title: "Transient"
category: userguide/api
date: 2017-07-24 16:04:40
disqus: 1
---

* TOC
{:toc}

Transient data is temporary, local data, i.e. data that does not get persisted or synchronized to
other clients, and disappears on reload. In regular web development, the DOM is by default
transient. Any changes made to the DOM disappears when the page is reloaded. If changes made to the
DOM needs to be persisted in regular development, specific effort has to be made to accomplish this.

With Webstrates, this is the other way around. By default, all changes made to the DOM get persisted
on the server and synchronized to all connected clients. If some data shouldn't get persisted,
special effort has to be made.

Webstrates allows the users to create transient data either through the custom HTML element
`<transient>` or through attributes prefixed with `transient-`. These are both configurable in the
[client configuration](/userguide/client-config.html#transient-elements-and-attributes).

## Transient elements

Any data added inside a `<transient>` tag simply doesn't get transferred to other clients:
```html
<html>
<body>
  This content will be saved on the server and synchronized to all connected users as per usual.
  <transient>
    This element and its contents will only be visible to the user who created the element.
  </transient>
</body>
</html>
```

This is especially useful for (temporarily) storing data received through signaling.

## Transient attributes

Any attribute name starting with `transient-` will automatically be transient:

```html
<div class="popupbox" transient-visible="true">Here's a box!</div>
```

The above element will be visible to all clients, but the attribute `transient-visible="true"` will
only be visible to the user who added it.

This is especially useful for managing CSS-targetable elements that should only be visible to
certain users:

```css
.popupbox {
  display: none;
}
.popupbox[transient-visible="true"] {
  display: initial;
}
```

<div class="warning box">
	<p><strong>A word of caution:</strong> If a user reloads a webstrate in which they have transient
	data, the transient data will be unrecoverable just as in a regular web application.</p>
</div>
