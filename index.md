---
layout: default
title: "Documentation"
---

# Webstrates

With Webstrates, webpages become collaboratively editable in real-time. Changes to the Document Object Model (DOM) of a page persist and are synchronised to all connected clients of the same page using [Operational Transformation](http://en.wikipedia.org/wiki/Operational_transformation) through [ShareDB](https://github.com/share/sharedb).

Webstrates can be used to develop interactive software where collaboration-support is the norm rather than the exception.

To see and read about some of the things it has been used for visit [webstrates.net](https://webstrates.net), or look at [our tutorials](/userguide/tutorials.html).

## Super quick demo!

Below you see two iframes transcluding the same webpage, namely:
<div id="url"></div>

Try modifying the text in one of the iframes now.

<style type="text/css" media="screen">
#url {
	font-size: 1.1rem;
	margin: .5rem 0;
	text-decoration: underline;
	text-align: center;
}
#iframeContainer {
	display: none;
}
#iframeContainer iframe {
	border: 1px solid #ccc;
	width: calc(50% - 12px);
	margin: 10px 0;
}

.fadein {
	display: initial !important;
	animation: 1.2s fadein;
}
@keyframes fadein {
	0% { opacity: 0; }
	100% { opacity: 1; }
}
</style>

<div id="iframeContainer">
	<iframe id="iframe1"></iframe>
	<iframe id="iframe2" style="margin-left:12px"></iframe>
</div>

Whoa! The changes got synchronized immediately, didn't they?

The document (or "webstrate") is running on our Webstrates server and basically just contains:
```html
<html>
<body contenteditable>
This is a <code>contenteditable</code> field. Write here!
</body>
</html>
```

And yet any changes to DOM get persisted and synchronized -- you can even inspect and modify the
DOM any way you like right now, and the changes will get persisted. Or you can link the address of
the page to a friend and start developing your own collaborative web application right now.

<script>
const urlField = document.getElementById('url');
const iframeContainer = document.getElementById('iframeContainer');
const iframe1 = document.getElementById('iframe1');
const iframe2 = document.getElementById('iframe2');

let webstrateId;
if (!location.hash) {
	const now = new Date().toISOString().substring(0, 10);
	const random = Math.random().toString(36).substring(2)
	webstrateId = `doc-${now}-${random}`;
	history.pushState(null, null, `#${webstrateId}`);
}
webstrateId = location.hash.substring(1);

const serverAddress = "hikaru.cs.au.dk";

urlField.innerHTML = `<a href="https://${serverAddress}/${webstrateId}/">` +
	`${serverAddress}/${webstrateId}/</a>`;

if (location.hash) {
	iframe1.src = `https://${serverAddress}/doc-prototype/release?copy=${webstrateId}`;
} else {
	iframe1.src = `https://${serverAddress}/doc-prototype/${webstrateId}`;
}

setTimeout(() => iframe2.src = `https://${serverAddress}/${webstrateId}/`, 300);

iframe2.onload = () => {
	iframeContainer.classList.add('fadein');
};
</script>
