---
layout: page
title: "Hello, world!"
category: gettingstarted
date: 2017-07-04 14:29:51
disqus: 1
---

* TOC
{:toc}

Creating the "Hello, World!" of Webstrates is very simple, so let's do it!

Start by pointing your web browser at [http://localhost:7007/](http://localhost:7007/). If
Webstrates has been [installed correctly](/gettingstarted/installation), you should now get
redirected to en empty page at [/frontpage](http://localhost:7007/frontpage).

To turn the frontpage into a "Hello, world!" example, go to your browser's Developer Tools.

<div class="info box">
	<h3>How to find your browser's Developer Tools</h3>
	<ul>
		<li>
			<strong>Apple Safari</strong>:
			Click Safari in the top-left corner, then choose <em>Preferences...</em>, go to the
			<em>Advanced</em> tab, check <em>Show Develop menu in menu bar</em> and close the popup
			window. Now go to the newly added <em>Develop</em> menu and choose <em>Show Web
			Inspector</em>.
		</li>
		<li>
			<strong>Google Chrome</strong>:
			Right-click on the webpage and choose <em>Inspect</em>.
		</li>
		<li>
			<strong>Mozilla Firefox</strong>:
			Click the Wrench icon in the Hamburger menu in the top-right corner, then choose <em>Toggle
			Tools</em>.
		</li>
		<li>
			<strong>Microsoft Edge</strong>:
			Click the More icon (<code>...</code>) in the top-right corner, then choose <em>Developer
			Tools</em>.
		</li>
	</ul>
</div>

In the DOM inspector, now right-click the `<body>` element and choose _Edit as HTML_. Modify the
line to read:

```html
<body><h1>Hello, world!</h1></body>
```

And click outside the text editing box. The text will now show up in the browser view and will be
persisted on the server. You will notice that if multiple browsers windows were pointed at
[/frontpage](http://localhost:7007/frontpage) when the change was made, the change would also be
reflected immediately in the other browser windows, too.

