---
layout: page
title: "Installation"
category: gettingstarted
date: 2017-05-30 23:28:16
disqus: 1
---

* TOC
{:toc}

Webstrates runs on Windows, macOS and Linux.

Before installing Webstrates, you'll need:

- [MongoDB](http://www.mongodb.org)
([installation guide](https://docs.mongodb.com/manual/installation/#tutorials)).
- [NodeJS](http://nodejs.org) ([installation guide](https://nodejs.org/en/download/package-manager/)) (version 8 or later).

To install:

- Clone [the Webstrates repository](https://github.com/Webstrates/Webstrates) with `git` or
[download a ZIP of the source code](https://github.com/Webstrates/Webstrates/archive/master.zip) and unzip it.
- Navigate to the repository root.
- If running Windows, modify the build command found in the scripts section of `package.json` to match:
```
SET NODE_ENV=production & webpack -p --config ./webpack.config.js
```
- Run the following from the root directory:
```
npm install --production # Installs required NPM packages
npm run build # Uses webpack to generate the client application code
npm start # Starts the Webstrates server
```

A local Webstrates server should now be running at
[http://localhost:7007/](http://localhost:7007/). By default, the server will be configured to use
basic authentication (see
[Server Config](/userguide/server-config.html#server-level-basic-authentication)). To log in, use
"web" as username and "strate" as password.

When navigating to [http://localhost:7007/](http://localhost:7007/), you should be redirected to
[http://localhost:7007/frontpage/](http://localhost:7007/frontpage/) automatically and be presented
with an empty webstrate (i.e. a white page).

The `--production` flag prevents development/testing packages from being installed. This speeds up
the installation and saves some space as  [puppeteer](https://www.npmjs.com/package/puppeteer)
(about 170 MB) won't be installed. It does, however, mean you won't be able to run the functional
tests.

## Advanced installation

Installing Webstrates as a system service on Linux behind Nginx with SSL support can be achieved by
following the guide
[Setup Webstrates using SSL on a Ubuntu 16.04 (Xenial Xerus)](https://github.com/Webstrates/Webstrates/wiki/Setup-Webstrates-using-SSL-on-a-Ubuntu-16.04-(Xenial-Xerus))
from the [Github Wiki](https://github.com/Webstrates/Webstrates/wiki).

Additionally, a script for setting up Webstrates on a single-board computer like a Raspberry Pi 3
can be found in [the Pocketstrates repository](https://github.com/Webstrates/pocketstrates).

## Upgrading

If the initial installation was done using `git`, run `git pull` to download the newest version. If
the initial installation was made by downloading a ZIP of the source code, download the most recent
version of the ZIP and extract it on top of the existing installation, overwriting any existing
files.

After upgrading the source, run `npm install` to install any potential new packages, followed by
`npm run build` to rebuild the client application code.