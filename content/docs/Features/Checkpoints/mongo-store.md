---
title: "Checkpoint in MongoDB"
linkTitle: "MongoDB"
weight: 5
description: >
    MongoDB checkpoint store
---

Although in many cases the file-based checkpoint works fine, storing the checkpoint outside the Replicator deployment is more secure. For that purpose you can use the MongoDB checkpoint store. This store writes the checkpoint as a MongoDB document to the specified database. It will unconditionally use the `checkpoint` collection, and will store the document using the unique id that must be provided in the settings. The uniqueness of the setting (`instanceId`) value is requires if you run multiple Replicator instances, and store checkpoints in the same Mongo database.

Here are the settings for the MongoDB checkpoint store:

```yaml
replicator:
  checkpoint:
    type: "mongo"
    path: "mongodb://mongoadmin:secret@localhost:27017"
    database: "replicator"
    instanceId: "test"
    checkpointAfter: 1000
```

Here, the `path` setting must contain a pre-authenticated connection string.

The `checkpointAfter` option defines the frequency of the stored checkpoint update. Smaller number increases the level of idempotency for the sink. For the highest level of idempotency this option should be set to `1`, so the checkpoint will be stored after writing each event. However, it also slows down the replication process, as storing the checkpoint to the database takes time, and increases the load on the MongoDB instance.
