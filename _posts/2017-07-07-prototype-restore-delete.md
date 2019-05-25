---
layout: page
title: "Prototype, Restorate and Delete"
category: userguide/api
date: 2017-07-06 16:10:08
disqus: 1
---

* TOC
{:toc}

## Prototyping

When creating a webstrate that's based on another webstrate, we call it _prototyping_. This can be done either through the [HTTP API](http://localhost:4000/userguide/http-api.html#creation-of-webstrates) or the JavaScript API.

### Prototype from a ZIP

To overwrite the existing webstrate with one from a ZIP archive (like one
[downloaded with `?dl`](http://localhost:4000/userguide/http-api.html#accessing-the-history-of-a-webstrate)),
you can prototype from a file:

```javascript
webstrate.newFromPrototypeFile([webstrateId])
```

Calling the above will open a file selector dialog that accepts ZIP files and returns a promise. The ZIP file
will then be extracted to the specified webstrateId. If the webstrate already exists or isn't empty, the promise will be rejected. If no webstrateId is specified, the current webstrateId will be assumed.

### Prototype from a URL

To prototype from a URL:

```javascript
webstrate.newFromPrototypeURL(url, [webstrateId])
```

The above will download the file (or page) from the specified URL and prototype it to the specified webstrateId. If the
webstrate already exists or isn't empty, the promise will be rejected. If no webstrateId is specified, the current
webstrateId will be assumed.

When using `webstrate.newFromPrototypeFile()`, the same rules apply as with the
[HTTP API](http://localhost:4000/userguide/http-api.html#creation-of-webstrates), so remember to add `?dl` to the
end of the URL if prototyping from another webstrate.

<div class="info box">
	When copying a webstrate that the copier doesn't have
	<a href="/userguide/permissions">permissions</a> to modify (read), one of two things will happen:
	<ul>
		<li>If the user is logged in, the read-write permissions are added to the permissions list
		of the copied webstrate. Note that the original permissions persist in the copy.</li>
		<li>If the user isn't logged in, the permissions will be wiped from the webstrate, so the
		webstrate uses the <a href="/userguide/permissions#defaults">default permissions</a>.</li>
	</ul>
	Note that it's not possible to copy a webstrate in which the user doesn't have read permissions,
	either explicitly in the <code>data-auth</code>, or implicitly by falling back to default
	permissions.
</div>

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