---
layout: page
title: "Protected Mode"
category: userguide/api
date: 2018-08-23 10:39:57
---

Protected mode allows developers to make the creation of all new elements or attributes transient.
This mode is especially useful when browser extensions or libraries continually pollute the DOM.
The protection from library code, however, is limited.

The mode is enabled by adding `data-protected` attribute to the HTML tag of the Webstrates **and reloading the page**.
When properly enabled, the console should show a message informing the user that the document is protected.

The existence of the the `data-protected` attribute will protect the persisted DOM both from the creation of attributes and from the creation of new DOM elements, the same as if `data-protected` was set to `all`.
However, the attribute can also be set to `elements` to only protect elements, and likewise it can be set to `attributes` to only protect from the creation of attributes, e.g.

```html
<html data-protected="attributes">
  ...
</html>
```

Naturally, it might still be useful to create non-transient elements in the DOM, even when element protection is enabled.
To create an element that's approved to be in the DOM, the element can be created with:

```javascript
document.createElement(tagName, { approved: true });
```

Likewise, for setting a non-transient attribute:

```javascript
Element.setAttribute(name, value, { approved: true });
```

Setting attributes with other APIs will automatically be approved (`dataset`, `classList`, `tabIndex`, `title`, etc.).

Whether the DOM is protected or not can be determined by looking at the global `webstrate.isProtected` property.
The property is false if the webstrate is not protected at all, and true if elements, attributes, or both are protected.
To know if only attributes or elements are protected, one will have to look at the `data-protected` property.
