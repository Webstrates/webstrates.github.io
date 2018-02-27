---
layout: page
title: "Versioning"
category: userguide
date: 2017-07-25 02:14:03
disqus: 1
---

* TOC
{:toc}

Webstrates uses the ShareDB backend, persisting and versioning every single change made to a
document, down to the keystroke. As a result, a webstrate can be
[restored](/userguide/api/restore-delete.html#restoring-a-webstrate) to any of its previous
versions, either by specifying a version number or a [tag](/userguide/api/tagging.html).

The operations history can be a accessed through `http://<hostname>/<webstrateId>?ops`, which is
also described in the
[HTTP API documentation](/userguide/http-api.html#accessing-the-history-of-a-webstrate).

## Example History

Let's create a new document, make it `contenteditable`, and then write "Hello, world!" in the DOM,
so we can check out the operation history.

If we start out by navigating to an empty document, say `http://<hostname>/my-test-webstrate`, we'll
be met with a DOM looking something like:

```html
<html><head><title>my-test-webstrate</title></head><body></body></html>
```

Inspecting the operations history will shows us something similar to:

```json
{
  "v": 0,
  "create": {
    "type": "http://sharejs.org/types/JSONv0",
    "data": [
      "html", { "__wid": "WeywP48a" },
      [
        "head", { "__wid": "prJTAVWx" },
        [
          "title", { "__wid": "8iqiqAix" },
          "my-test-webstrate"
        ]
      ],
      [
        "body", { "__wid": "mBTnnTpj" }
      ]
    ]
  },
}
```

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

At this point, we will be at version 0 (evident by the `v` property on the object). The version
can also be accessed by calling `version` on the global webstrate object:

```json
> webstrate.version
0
```

Or by accessing `http://<hostname>/my-test-webstrate?v`:

```json
{ "version": 0 }
```

After adding `contenteditable` to the body, so we can write in the document, and adding
"Hello, World!" to the document, the document will now look something like:

```html
<html><head><title>my-test-webstrate</title></head><body contenteditable="">Hello, world!</body></html>
```

And then operations history will have expanded to now include:

```json
[
  ...
  { "v": 1, "op": [ {"oi": "", "p": [3, 1, "contenteditable"] } ] },
  { "v": 2, "op": [ {"li": "H", "p": [ 3, 2 ] } ] },
  { "v": 3, "op": [ {"si": "e", "p": [ 3, 2, 1 ] } ] },
  { "v": 4, "op": [ {"si": "l", "p": [ 3, 2, 2 ] } ] },
  { "v": 5, "op": [ {"si": "l", "p": [ 3, 2, 3 ] } ] },
  { "v": 6, "op": [ {"si": "o", "p": [ 3, 2, 4 ] } ] },
  { "v": 7, "op": [ {"si": ",", "p": [ 3, 2, 5 ] } ] },
  { "v": 8, "op": [ {"si": " ", "p": [ 3, 2, 6 ] } ] },
  { "v": 9, "op": [ {"si": "W", "p": [ 3, 2, 6 ] } ] },
  { "v": 10, "op": [ {"si": "o", "p": [ 3, 2, 8 ] } ] },
  { "v": 11, "op": [ {"si": "r", "p": [ 3, 2, 9 ] } ] },
  { "v": 12, "op": [ {"si": "l", "p": [ 3, 2, 10 ] } ] },
  { "v": 13, "op": [ {"si": "d", "p": [ 3, 2, 11 ] } ] },
  { "v": 14, "op": [ {"si": "!", "p": [ 3, 2, 12 ] } ] }
]
```

The first operation in the above represents the insertion of the `contenteditable` attribute (`oi`
stands for object insertion), and the remaining operations each correspond to a key press (`li`
for list insertion and `si` for string insertion).

As is also evident from the operations history, each key press has been given its own version
number. We will now be at version 15:

```json
> webstrate.version
15
```

Note that we now are at version 15, even though the latest operation is labeled version 14. This is
because the version attached to the operation actual indicates the version _prior_ to the operation,
just as the _first_ version (the creation) of the document also said version 0.

## Restoring a document

Part of the Webstrates philosophy is to document all changes, the operations history being an
example of that. As such, reverting a webstrate doesn't actually delete any operations.

Instead, the differences between the JSON representation of the document at the current version and
the desired version is calculated using the
[`json0-ot-diff`](https://www.npmjs.com/package/json0-ot-diff) library which returns the differences
as an array of operations. These operations are then applied to the document.

Continuing from the example above,
[restoring](/userguide/api/restore-delete.html#restoring-a-webstrate) to version 7 -- the version
right after having finished writing "Hello" -- will add just two new operation to the history:

```json
[
  ...
  { "v": 15, "noop": "documentRestore", },
  { "v": 16, "op": [ {"p": [3, 2 ], "ld": "Hello, World!", "li": "Hello" } ] }
]
```

The first operation being a dummy operation (`noop` meaning "no operation") just to indicate that
the document was restored.

The second operation here has two parts, first a list deletion (`ld`) of the previous string
("Hello, World!"), followed by a list insertion (`li`) of the desired string "Hello".

After the restoration, the document will now be back to:

```html
<html><head><title>my-test-webstrate</title></head><body contenteditable="">Hello</body></html>
```

The important thing to note, however, is that the document's version will _not_ be 7, as we restored
to, but rather at 17. We added the two operations created (the `noop` and the `op`) to the document
history, bringing us from 15 to 17.

Restoring a document is thus non-destructive to the history. Another way of restoring the document
could have been to remove all operations after version 7 and rebuilt the document up to that point,
but in that case, anything after version 7 would have been lost. Using our approach, it is now
possible to "go back to the future", i.e. restore version 15 again:

```json
> webstrate.version
17
> webstrate.restore(15)
undefined
> webstrate.version
19
```

Which would again give us a document containing just "Hello, World!".