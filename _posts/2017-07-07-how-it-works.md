---
layout: page
title: "How it works"
category: developerguide
date: 2017-07-07 16:07:08
disqus: 1
---

* TOC
{:toc}

Webstrates' API provides a variety of functionality, but the main functionality that Webstrates
resolves around is DOM synchronization.

Webstrates' DOM synchronized is built around 4 components, namely [JsonML](http://www.jsonml.org),
the [`MutationObserver` JavaScript API](https://developer.mozilla.org/en/docs/Web/API/MutationObserver),
Operational Transformations, and [ShareDB](https://github.com/share/sharedb).

## JsonML
JsonML (or JSON Markup Language) is language for representing XML-based markup as JSON. For
instance, the following HTML:

```html
<ul>
  <li style="color: red">First Item</li>
  <li><a href="/" style="color: green">Second Item</a></li>
</ul>
```

Could in JsonML look like:

```json
"ul",
{},
[
  "li", { "style": "color: red" }, "First Item"
],
[
  "li",
  {},
  [
    "a", { "href": "/", "style": "color: green" }, "Second Item"
  ]
]
```

All webstrates are persisted as JsonML and are converted back to HTML before being presented to the
user in the browser.

## The `MutationObserver` JavaScript API
The [`MutationObserver` JavaScript API](https://developer.mozilla.org/en/docs/Web/API/MutationObserver)
provides a way for developers to react to changes in the DOM. Every change to the DOM (except
changes to input fields and textareas as several other mechanism for detecting changes to these
elements exist) triggers a user-defined callback function, allowing developers to track changes
made by other parts of a web application.

In Webstrates, we listen for these changes and calculate the differences (or "deltas") between the
previous DOM state and the new DOM state after the mutation has occurred. We express these
differences as Operational Transformation operations.

## Operational Transformations

Operational Transformations (OT) is a standard for expressing differences between many types of
documents. With Webstrates, we use the [json0](https://github.com/ottypes/json0) standard to
represent our HTML differences. These differences are called operations and are intended to be
applied to a document to take it from one state to another, like taking a JsonML representation of a
webstrate prior to a mutation to a JsonML representation of the same webstrate after the mutation
without needing to calculate (and transmit) an entirely new JsonML representation of the DOM.

For instance, if we're looking at the HTML document from before and we'd like to change the text
color of "Second Item" by replacing `green` with `blue` in the `style` attribute, this changes (or
operations) could look something like:

```json
[
  {
    "sd": "green",
    "p": [ 3, 2, 1, "style", 7 ]
  },
  {
    "si": "blue",
    "p": [ 3, 2, 1, "style", 7 ]
  }
]
```

The first operation here tells us to perform a string deletion (`sd`) from the path at
`[3, 2, 1, "style", 7]`, meaning the 3rd child, then that child's 2nd child, that child's 1st child,
and the `style` property of that object. From that object, we delete the word `green` starting at
7th characters in. (i.e. after `color: `).

The second operation is very similar, but this is instead a string insertion (`si`) of `blue` again
starting at the 7th character of the property's value.

However, just keeping a local JsonML representation of an HTML file up-to-date doesn't provide much
collaborability. For that, we need ShareDB.

## ShareDB

[ShareDB](https://github.com/share/sharedb) is a real-time, fully-versioned JSON database operated
on with `json0` type operational transformations.

For Webstrates, we can therefore use it to store our JsonML documents and submit any changes
detected in the DOM by our `MutationObserver` through OT operations. ShareDB will afterwards make
sure to keep the JsonML representations in the clients consistent.

Just as the `MutationObserver` JavaScript API provides ways of detecting changes to the DOM, ShareDB
also provides ways of detecting changes to the JsonML document. In Webstrates, we listen for these
changes (also represented as OT operations) and convert them to their equivalent DOM changes, thus
providing us with a synchronized, consistent two-way binding between the DOM and ShareDB's shared
JsonML representation of the DOM across clients.

For a more in-depth understanding of how Webstrates work, check out the rest of the Developer guide,
as well as the [source code for Webstrates](https://github.com/Webstrates/Webstrates) and
[the source code for ShareDB](https://github.com/share/sharedb), both readily available on GitHub.