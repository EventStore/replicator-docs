---
title: "Checkpoints"
linkTitle: "Checkpoints"
weight: 5
description: >
  Supported checkpoint stores
---

The location of the last processed event in the source is known as _checkpoint_. Replicator supports storing checkpoints in different stores, but nly one store can be used at a time. If you want to run the replication again, from the same source, using the same Replicator instance, you need to delete the checkpoint from the store.
