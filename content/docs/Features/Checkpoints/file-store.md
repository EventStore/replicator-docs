---
title: "Checkpoint file"
linkTitle: "File system"
weight: 5
description: >
    File system checkpoint store
---

By default, Replicator stores the checkpoint in a local file. For that purpose, for example, the Helm chart has a persistent volume claim template, which is mapped to the `/data` path. The file name is `checkpoint` by default.

Here is the default configuration used by the Helm chart:

```yaml
replicator:
  checkpoint:
    type: file
    path: "/data/checkpoint"
    checkpointAfter: 1000
```

The `checkpointAfter` option defines the frequency of the stored checkpoint update. Smaller number increases the level of idempotency for the sink. For the highest level of idempotency this option should be set to `1`, so the checkpoint will be stored after writing each event. However, it also slows down the replication process, as storing the checkpoint to the file takes time.
