---
layout: page
title: "Activity History"
category: userguide/api
date: 2019-03-25 11:54:44
---

Webstrates keeps track of which documents users have been active in (i.e. created or modified), and
not only through the op log. Webstrates also keeps a log of the last time any user modified any
webstrate and makes this information available through the method
`webstrate.user.history([options])` which returns a promise.

The `options` parameter is an optional object that (currently) allows the user to specify a limit
to how many webstrates to query for. When limiting, the returned webstrates will always be the most
recently modified. The default limit is 50.

A call with a limit of 5 will look like:

```javascript
await webstrate.user.history({ limit: 5 });
```

And will give back the user an object that looks something like:

```javascript
{
  angry-kangaroo-24: "2019-03-01T15:00:24.000Z"
  cuddly-kangaroo-97: "2019-03-20T15:59:09.906Z"
  friendly-kangaroo-22: "2019-03-01T14:39:25.000Z"
  massive-kangaroo-91: "2019-03-01T14:39:29.000Z"
  weak-kangaroo-65: "2019-03-20T16:13:19.585Z"
}
```

To get back the entire history, one may just specifiy an appropriately large `limit`, for instance
`{ limit: Number.MAX_SAFE_INTEGER }`.

## Creating Activity History retroactively

When updating Webstrates from a previous version without the Activity History feature, old documents
will not have been added to the activity history, and as a result will not show up in the results
returned by `webstrates.user.history()`. To build the activity history retroactively,
the below script can be executed from the Webstrates root directory.

```javascript
const MongoClient = require('mongodb').MongoClient;
const ObjectID = require('mongodb').ObjectID;

global.APP_PATH = __dirname;
const configHelper = require(APP_PATH + '/helpers/ConfigHelper.js');
const config = global.config = configHelper.getConfig();

const printProgress = (i, total) => {
  process.stdout.cursorTo(0);
  process.stdout.write(`Progress: Op ${i} of ${total} (${(i / total * 100).toFixed(2)}%)`);
};

const sessionMap = new Map();

MongoClient.connect(global.config.db, async (err, db) => {

  const userHistory = db.collection('userHistory');

  const sessions = await db.collection('sessionLog').find({
    userId: { $ne: 'anonymous:' }
  }, {
    userId: 1, sessionId: 1
  });

  while (await sessions.hasNext()) {
    const session = await sessions.next();
    sessionMap.set(session.sessionId, session.userId);
  }

  const ops = await db.collection('o_webstrates').find({}, {
    _id: 1, d: 1, src: 1
  });

  const TOTAL_OP_COUNT = await ops.count();
  let i = 0;

  while (await ops.hasNext()) {
    const op = await ops.next();
    ++i;

    const userId = sessionMap.get(op.src);
    if (!userId) continue;

    const timestamp = ObjectID(op._id).getTimestamp();
    const webstrateId = op.d;

    const $set = {};
    $set[`webstrates.${webstrateId}`] = timestamp;

    userHistory.update({ userId }, { $set }, { upsert: true }, (err, res) => {
      if (err) console.error(err);
    });

    printProgress(i, TOTAL_OP_COUNT);
  }

  printProgress(i, TOTAL_OP_COUNT);
  process.exit();
});
```

After having run the script, the activity history will be up date and stay that way. If the script
gets interrupted, the collection accidentally wiped, or the user wishes to rerun the script for any
reason, it is safe to do so.

The script can be run while the Webstrates server is running, but keep in mind that on exceptionally
large databases, the script may take as long as 20 minutes to finish and performance might be
slightly on the Webstrates server during this time period.