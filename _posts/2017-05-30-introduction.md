---
layout: page
title: "Introduction"
category: developerguide
date: 2017-05-30 23:30:31
disqus: 1
---

<div class="box warning">
	<p><strong>This is the developer's guide</strong> for people interested in helping develop the Webstrates platform, or expand it with functionality for their own use.
	If you just want to develop applications on top of the platform, this is most likely not what you're looking for.
	</p>
</div>

* TOC
{:toc}

The Webstrates client code is made using the [webpack](https://webpack.github.io/) module bundler.
As a result, the client code consists of about 30 modules that interact with each other through
events.

Building the client code around events allows the system to have clear separation of concern while providing extensive expandability by attaching more event listeners to existing events or adding new events altogether.

There are two types of modules, core modules and (non-core) optional modules.
The core modules are essential for the client to work as they explicitly depend on each other's events.
The optional modules, on the other hand, can be disabled without breaking the core system.
As some optional modules depend on each other, however, disabling an optional module may also break another optional module that depends on it.

As an example, the `coreMutation` module (which manages [MutationObservers](/developerguide/how-it-works.html#the-mutationobserver-javascript-api)) emits events to the client system whenever changes to the DOM occurs.
These events are emitted through `coreEvents` -- the module for managing and dispatching events -- and received by `coreOpCreator`.
The `coreOpCreator` module is responsible for creating [OT operations](/developerguide/how-it-works.html#operational-transformations) and emitting these operations to the system where they will
finally be picked up by the `coreDatabae` module, which submits the operations to the [ShareDB backend](/developerguide/how-it-works.html#sharedb).
Disabling any of these modules would break the core system.

On the other hand, an optional module like `assets` could be disabled, leaving the remaining system intact, only removing the [assets functionality](/userguide/api/assets.html) from the client.

All events fired on

Modules can be disabled by removing them from [the client configuration](/userguide/client-config.html#modules)'s modules section. As core modules cannot be disabled, they are not
present in the config's modules section.

## List of Modules

| Module name         | Description
|---------------------|-----------------------------------------------------------------------------
| `coreDatabase`      | [ShareDB](/developerguide/how-it-works.html#sharedb) wrapper/adapter.
| `coreEvents`        | Event handler. Handles system (inter-modular) events.
| `coreJsonML`        | Library for converting between [JsonML](/developerguide/how-it-works.html#jsonml) and HTML (DOM) through `coreJsonML.toHTML()` and `coreJsonML.fromHTML()`.
| `coreMutation`      | Uses [MutationObservers](/developerguide/how-it-works.html#the-mutationobserver-javascript-api) to listen for DOM changes and emits these through `coreEvents`.
| `coreOpApplier`     | Applies operations received through `coreEvents` to the DOM. These events are usually emitted by `coreDatabase`.
| `coreOpCreator`     | Creates operations based on mutations received from `coreMutation`.
| `corePathTree`      | Library for creating and attaching PathTrees to DOM nodes.
| `corePopulator`     | Populates the DOM with the webstrate once it has been retrieved by `coreDatabase`.
| `coreUtils`         | Library containing a variety of useful functions for creating random strings, cloning objects, etc.
| `coreWebsocket`     | Websocket wrapper that allows other modules access to the underlying websocket.
| `assets`            | Provides userland asset functionality (e.g. `webstrate.assets` and the userland `asset` event).
| `clientManager`     | Provides userland client functionality (e.g. `webstrate.clients` and the userland `clientJoin` and `clientPart` events).
| `connectionEvents`  | Provides connection events, system and in userland (i.e. `connect`, `disconnect`, `reconect`).
| `cookies`           | Provides userland cookie functionality (i.e. `webstrate.cookies` and the userland `cooieUpdateHere` and `cookieUpdateAnywhere` events).
| `domEvents`         | Providues userland DOM events (e.g. `attributeChanged`, `nodeAdded`, `nodeRemoved*`, `insertText`, etc.).
| `globalObject`      | Creates the global object (`window.webstrate`) which other modules often use to expose their own functionality in userland.
| `keepAlive`         | Simply sends keep alive messages over the websocket to ensure it doesn't terminate prematurely.
| `loadedEvent`       | Creates and fires the userland event `loaded`, as well as the `loadedTriggered` system event.
| `messages`          | Provides userland messaging functionality (e.g. `webstrate.messages` and the userland event `messageReceived`).
| `nodeObjects`       | Creates the `webstrate` object on every node in the DOM, allowing users to attach Node-specific event listeners (e.g. `attributeChanged`, `nodeAdded*`, etc.).
| `permissions`       | Provides users with permissions functionality (e.g. `webstrate.permissions` and the system event `globalPermissions` and `userPermissions`). Note that disabling this module will not allow unauthorized access to restricted webstrates.
| `signalStream`      | Provides userland signal streaming functionality.
| `signaling`         | Provides userland signaling functionality e.g. `Node.signal` as well as the `signal` event on nodes and the global object).
| `tagging`           | Provides userland tagging functionality (e.g. `webstrate.tag` as well as the `tag` event on the global object).
| `transclusionEvent` | Creates and fires the userland event `transcluded` when a webstrate has been transcluded.
| `userObject`        | Creates the user object (i.e. `webstrate.user`) containing user information, and upon which other modules depend (like `cookies`).
|---------------------|-----------------------------------------------------------------------------

See [Creating a module](/developerguide/creating-a-module.html) to learn how to create a module.

## System Events

Below is a list of all system (i.e. inter-modular) events created by the Webstrates client system.
Any module can listen for and react to these events.

|-----------------------------|----------------------------------|----------------------------------
| Event name                  | Emitted by                       | Used by
|-----------------------------|----------------------------------|----------------------------------
| `asset`                     | `assets`                         |
| `clientJoin`                | `clientManager`                  |
| `clientPart`                | `clientManager`                  |
| `clientsReceived`           | `clientManager`                  |
| `connect`                   | `connectionEvents`               |
| `disconnect`                | `connectionEvents`               |
| `reconnect`                 | `connectionEvents`               |
| `createdOps`                | `coreOpCreator`                  | `permissions`, `tagging`, `coreDatabase`
| `DOMAttributeRemoved`       | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `DOMAttributeSet`           | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `DOMAttributeTextDeletion`  | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `DOMAttributeTextInsertion` | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `DOMNodeDeleted`            | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `DOMNodeInserted`           | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `DOMTextNodeDeletion`       | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `DOMTextNodeInsertion`      | `coreOpCreator`, `coreOpApplier` | `domEvents`
| `globalPermissions`         | `permissions`                    |
| `loadedTriggered`           | `loadedEvent`                    | `clientManager`, `transclusionEvent`, `messages`
| `messageReceived`           | `messages`                       |
| `messageSent`               | `messages`                       |
| `mutation`                  | `messages`                       |
| `populated`                 | `corePopulator`                  | `loadedEvent`, `nodeObjects`, `signalStream`
| `receivedDocument`          | `coreDatabase`                   | `tagging`
| `receivedOps`               | `coreDatabase`                   | `copreOpApplier`, `permissions`, `tagging`
| `receivedSignal`            | `signaling`                      |
| `receivedTags`              | `tagging`                        |
| `userObjectAdded`           | `userObject`                     |
| `userPermissions`           | `permissions`                    |
| `webstrateObjectAdded`      | `nodeObjects`                    | `domEvents`, `signaling`, `signalStream`, `transclusionEvent`
| `webstrateObjectsAdded`     | `nodeObjects`                    | `domEvents`, `signaling`, `signalStream`, `transclusionEvent`
|-----------------------------|----------------------------------|----------------------------------