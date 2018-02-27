---
layout: page
title: "Server Configuration"
category: userguide
date: 2017-07-07 00:26:06
disqus: 1
---

* TOC
{:toc}

Webstrates ships with a sample server configuration file
([`config-sample.json`](https://github.com/Webstrates/Webstrates/blob/master/config-sample.json))
that gets copied over to `config.json` if no `config.json` file is present when Webstrates starts.

## Basic settings

|--------------------|--------------------------------------------|---------------------------------------|
| Property           | Description                                | Default                               |
|--------------------|--------------------------------------------|---------------------------------------|
| `listeningAddress` | IP address to listen on.                   | `0.0.0.0` (all addresses)             |
| `listeningPort`    | Port to listen on.                         | `7777`                                |
| `db`               | MongoDB connection url.                    | `mongodb://127.0.0.1:27017/webstrate` |
| `maxAssetSize`     | Max file size for assets in MB.            | `20`                                  |
| `maxAge`           | How long static files should be cached (not assets). Defined in milliseconds or using [ms](https://www.npmjs.com/package/ms) format. | `1d` (24 hours) |
| `rateLimit`        | [Configuration object for rate limiting](#rate-limiting) | See [`config-sample.json`](https://github.com/Webstrates/Webstrates/blob/master/config-sample.json). |
| `basicAuth`       | [Configuration object for basic auth](#server-level-basic-authentication) | See [`config-sample.json`](https://github.com/Webstrates/Webstrates/blob/master/config-sample.json). |
| `pubsub`           | Reddis connection url (multithreading).    | unset                                 |
| `threads`          | Number of threads to use (multihtreading). | unset                                 |
| `niceWebstrateIds` | Whether to use random strings as webstrateIds or more human readable random names. | `false` |
|--------------------|--------------------------------------------|---------------------------------------|

## Rate limiting

To avoid having clients DoS'ing the server unintentionally by having faulty Webstrates application
code, sending thousands of messages to the server per second, Webstrates allows for rate limiting
the number of requests.

Operations are generally a lot heavier on the server than other kinds of messages (keep alive,
signals, cookie changes, user messages), so these can be configured separately.

- `opsPerInterval` defines the number of operations allowed within the defined interval.
- `signalsPerInterval` defines the number of signals (and other kind of messages) allowed within
the defined interval.
- `intervalLength` defines the interval i milliseconds.
- `banDuration` defines the ban duration in milliseconds.

A decent configuration may be something like below, which allows 500 operations and 2000 signals
over a 15 second period:

```json
"rateLimit": {
  "opsPerInterval": 500,
  "signalsPerInterval": 2000,
  "intervalLength": 15000,
  "banDuration": 30000
}
```

With the above, the server will disconnect and ban any client (by IP) for 30 seconds if they exceed
the limitation.

## Authentication

### Server level basic authentication
To enable basic HTTP authentication on the Webstrates server, add the following to `config.json`:

```json
"basicAuth": {
  "realm": "Webstrates",
  "username": "some_username",
  "password": "some_password"
}
```

### Read-write and read-only permissions per webstrate
To setup read only permissions to specific webstrates or users, OAuth is required. We recommend using [GitHub](https://github.com) as authentication provider as Webstrates already ships with the required NPM packages.

To get started, [register an OAuth application with GitHub](https://github.com/settings/applications/new).

On the register page, _Homepage URL_ should be set to `http://localhost:7007/`, assuming the system is running locally on port 7007 (as per default). If other users need to be able to use the OAuth authentication system as well, the _Homepage URL_ should be set to the external address.

_Auhtorization callback URL_ should be set to `http://localhost:7007/auth/github/callback`.

After clicking _Register Application_, you'll be presented with a _Client ID_ and _Client Secret_, which needs to be added (along with other information) to your server `config.json` file:

```json
"auth": {
  "providers": {
    "github": {
      "node_module": "passport-github",
      "config": {
        "clientID": "<github client id>",
        "clientSecret": "<github secret>",
        "callbackURL": "http://<hostname>/auth/github/callback"
      }
    }
  }
}
```

### Logging in with OAuth/GitHub

To log in to the webstrates server, navigate to [http://localhost:7007/auth/github](http://localhost:7007/auth/github), sign in and authorize. After successful login, you'll be redirected to [http://localhost:7007/frontpage](http://localhost:7007/frontpage). To confirm you've actually been logged in, writing `webstrate.user` in the browser console should yield a user object with your username (among other information).

### Managing permissions

Access rights are added to a webstrate as a `data-auth` attribute on the `<html>` tag in JSON format:

```html
<html data-auth='[{"username": "cklokmose", "provider": "github", "permissions": "rw"},
  {"username": "anonymous", "provider": "", "permissions": "r"}]'>
...
</html>
```

The above example provides the user with GitHub username _cklokmose_ permissions to read and write (`rw`), while users who are not logged in (`anonymous`) only have read (`r`) access.

### Default permissions

It is also possible to set default permissions. Adding the following under the `auth` section in `config.json` will apply the same permissions as above to all webstrates without a `data-auth` property.

```json
"defaultPermissions": [
  {"username": "cklokmose", "provider": "github", "permissions": "rw"},
  {"username": "anonymous", "provider": "", "permissions": "r"}
]
```

### Permission timeout

To increase performance, permissions are cached in the system. When the `data-auth` property of a
webstrate is modified, the cache is invalidated. However, as a failsafe, permissions also time out
after a set period of time. This can be configured with `auth.permissionTimeout`, which defines the
number of seconds permissions can be remain cached before they need to be revalidated. (default
300 seconds).

### Cookie

Webstrates uses cookies to save the user's credentials. The cookie's name (`cookieName`), encryption
key (`secret`) and life time (`duration`) can be configured. The `duration` is defined in
milliseconds and defaults to 31536000000 (365 days).

While the default `cookieName` and `duration` are fine, the `secret` _should_ be changed by the user
before running Webstrates! Any random text string of 30+ characters is sufficient.

```json
"cookie": {
  "secret": "This is a secret",
  "cookieName": "session",
  "duration": 31536000000
}
```

## Multi-threading

To enable multithreading, Redis must be installed (to be used for publish/subscribe between
threads) and the config must contain to properties, namely:

- `pubsub` which must be set to the Redis connection url, and
- `threads` which must contain the number of desired threads.

For example:

```json
  "pubsub": "redis://localhost:6379/",
  "threads": 4
```

If `threads` is set to 0, Webstrates will automatically create one thread for each CPU core.

## Tagging

[Auto-tagging](/userguide/api/tagging.html#auto-tagging) can also be configured. Currently, only
very basic configuration is possible, namely:

- `autoTagInterval` which defines how long a webstrate must have been "inactive" (without ops) for
it to become autotagged on next action (default 3600 seconds (60 minutes)).
- `tagPrefix` defines the prefix of the tag that'll be created (default "Session of ").

```json
  "tagging": {
    "autotagInterval": 3600,
    "tagPrefix": "Session of "
  },
```

Tags are postfixed with the current date time, so a tag might look something like:
"Session of Thu Jun 29 2017 14:03:30 GMT+0200 (CEST)".

## WebstrateIds and `niceWebstrateId`

When navigating to `/new` to create a new webstrate, when using `?copy` to copy a new webstrate, or
when prototyping a new webstrate, Webstrate automatically generates a random webstrateId, unless one
has been specified.

By default, this webstrateId will be a random string betweeen 7-14 characters using
`A-Z, a-z, 0-9, _-`, such as `S1KeySFzM`. When `niceWebstrateId` is set to `true`, webstrateIds
that are easier to share will instead be used. These will be on the form
\<verb\>-\<animal\>-\<number\>, e.g. `old-kangaroo-26`.