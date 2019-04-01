---
layout: page
title: "Permissions"
category: userguide/api
date: 2018-08-27 19:15:14
---

* TOC
{:toc}


Webstrates supports per-webstrate permission configuration, allowing users of a webstrate to only
permit certain users read and/or write access to a webstrate.

To enable the permissions functionality, the Webstrate server must be configured with at least one
OAuth provider.
See [Server Configuration](/userguide/server-config.html#read-write-and-read-only-permissions-per-webstrate) for information on how to configure your server with permissions.

Permissions in a webstrate is configured using the `data-auth` attribute that can be set on the `<html>` tag in JSON format.

## The `data-auth` property

If no `data-auth` attribute has been set (or it's malformed), any user will have access to the webstrate according to the [default permissions defined in the server configuration](/userguide/server-config.html#default-permissions).
This will usually mean access to view, modify or delete the webstrate, unless
[`loggedInToCreateWebstrates` has been configured in the server configuration](/userguide/server-config.html#logged-in-to-create).
In this case, only users logged in with the configured OAuth providers will be able to modify or create webstrate documents.

The `data-auth` should contain a serialized JavaScript array, consisting of objects with the properties `username`, `provider` and `permissions`, or alternatively, objects containing only `webstrateId` to inherit permissions from another webstrate.

```html
<html data-auth='[{"username": "cklokmose", "provider": "github", "permissions": "rw"},
  {"username": "anonymous", "provider": "", "permissions": "r"}]'>
...
</html>
```

The above example provides the user with GitHub username _cklokmose_ permissions to read and write (`rw`), while users who are not logged in (_anonymous_) only have read (`r`) access.
Logged-in users will automatically inherit the permissions of the _anonymous_ user (if defined), and thus also have read permissions to the webstrate.
Any users not present in the permissions list will not have any access to the webstrate (either explicitly or implicitly through the anynomous user).

Any user with write permissions will implictly also have read permissions.
Adding `r` to _cklokmose_ in the above is thus technically redundant, but it adds some clarity to the user.

<div class="info box">
  <strong>Wait, what's my username and provider?</strong>
  <p>Each user logged in with an OAuth provider will have a unique <code>userId</code>.
  The <code>userId</code> is the username and provider joined together with a <code>:</code>.</p>
  <p>When logged in (authorized with a OAuth provider), the <code>userId</code>, <code>username</code>,
  and <code>provider</code> will be available on the <code>webstrate.user</code> object as
  <code>webstrate.user.userId</code>, <code>webstrate.user.username</code>,
  and <code>webstrate.user.provider</code>, respectively.</p>
  <p> The <code>webstrate.user</code> will also have a <code>permissions</code> property, containing
  the user's permissions in the current webstrate, e.g. <code>rw</code>.</p>
</div>

## Admin permissions

The admin permission `a` allows only certain users to be able to edit the permissions of a webstrate and delete and restore the webstrate.


```javascript
[ {"username": "cklokmose", "provider": "github", "permissions": "rw"},
  { "username": "kbadk", "provider": "github", "permissions": "arw" },
  ... ]
```

In the above example, the user _kbadk_ has been promoted to admin of the document and therefore neither _cklokmose_ or any other user may modify the `data-auth` property or delete or restore the webstrate.

Deleting and restoring the webstrate is restricted, because otherwise a malicious user may simply delete the webstrate and recreate it as their own, or restore the webstrate to before the permissions were added and take over the webstrate.

A webstrate can have multiple admins. Admin privilege is not inherited.

## Permission inheritance

Permission inheritance allows one webstrate to inherit all its permissions from another webstrate, thus making it easier to manage users across multiple webstrates.

Adding `{ webstrateId: <some-webstrateId> }` to the permissions list (`data-auth`), inherits the permissions of `<some-webstrateId>` into the current webstrate.


```javascript
[ {"username": "raedle", "provider": "github", "permissions": "rw"},
  { "webstrateId": "test-webstrate" },
  ... ]
```

In the above example, we inherit the permissions from the webstrate `test-webstrate`.
If `test-webstrate` contains the permissions we defined in the previous example, the users _cklokmose_
and _kbadk_ would now also have `rw` to the document.

Note that the admin privilege is not inherited into the new document: The user _kbadk_ will therefore only have `rw` permission in the webstrate; the same as _raedle_ and _cklokmose_.

Recursive inheritance is supported up to a depth of 3. E.g. webstrate _X_ can inherit from _Y_, _Y_ can inherit from _Z_, but even though _Z_ inherits from _W_, the users of _W_ won't have access to X.
Both the users of _Y_ and _Z_, however, will have access.

Inheriting from multiple webstrates is also allowed.

If multiple permissions are found for the same user in different webstrates inherited from, the first one found takes precedence. E.g. if user A has `r` permissions to webstrate _Y_ and `rw` permissions to webstrate _Z_, and _X_ inherits from _Y_ and then from _Z_ (i.e. `[{ webstrateId: 'Y' }, { webstrateId: 'Z' }, ...]`), user _A_ will only have `r` permissions to _X_. This mechanism can also be used to exclude members who would otherwise have access by simply preceding the permissions list with an empty permission, e.g. `[{ username: 'A', provider: 'github', permissions: '' }, { webstrateId: 'Y' }, { webstrateId: 'Z' }, ...]`. This will disallow user _A_ permission to _X_, regardless of _A_'s permissions in _Y_ and _Z_.

If webstrate _X_ inherits from webstrate _Y_, and permissions in _X_ changes, they will immediately be reflected (as usual). However, if the permissions change in Y, the changes will not be reflected until the permissions of X have been modified, or until the permission cache expires.
Permissions will expire after 2 minutes or as [defined by `permissionTimeout` in the server configuration](/userguide/server-config.html#permission-timeout).