---
layout: page
title: "Restoration and Deletion"
category: userguide/api
date: 2017-07-06 16:10:08
disqus: 1
---

* TOC
{:toc}

## Restoring a webstrate

Webstrates has fine-grained [versioning control](/userguide/versioning.html), saving every keystroke
in the operations history. Restoring a document to a previous version can be done either through the
[HTTP API](/userguide/http-api.html#restoring-a-webstrate):

* GET on `http://<hostname>/<webstrateId>?restore=<versionOrTag>` restores the document to look like
it did in version or [tag](/userguide/api/tagging.html) `<versionOrTag>` and redirects the user to
`/<webstrateId>`. This will apply operations on the current verison until the desired version is
reached and will therefore not break the operations log or remove from it, but only add additional
operations.

Or through the JavaScript API by calling `webstrate.restore(versionOrTag, fn)`. `fn` is
a function callback that takes two arguments, a potential error (null on success) and the new
version:

```javascript
webstrate.restore(versionOrTag, function(error, newVersion) {
  if (error) {
    // Handle error.
  } else {
    // Otherwise, document is now at newVersion, identical to
    // the document at versionOrTag.
});
```

`webstrate.restore` returns a new version, because when restoring a document, the history is not
actually reverted, rather appended to. A bunch of operations is applied to the document to make the
document identical to what it was at the requested version.

To find out more about why and how it works, check out the
[Versioning](/userguide/versioning.html#restoring-a-document) page.

## Deleting a webstrate

Deleting a webstrate can exclusivel be done through the [HTTP API](http://localhost:4000/userguide/http-api.html#deleting-a-webstrate):

* GET on `http://<hostname>/<webstrateId>?delete` will delete the document and redirect all
connected users to the server root. The webstrate data including operations history, tags, assets,
etc. will all be completely and unrecoverably deleted.

 This versioning can be accessed through
the HTTP API, i.e. `?ops` and
accessing previous versions, see also tags for marking versions with memorables names.