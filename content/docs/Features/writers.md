---
title: "Write modes"
linkTitle: "Write modes"
date: 2021-04-05
weight: 5
description: >
    Different write modes to support string ordering guarantee for `$all`, or relax it for better performance.
---

## Write modes

Replicator will read events from the source cluster using batched reads of 4096 (default) events per batch. As it reads from `$all`, one batch will contain events for different streams. Therefore, writing events requires a single write operation per event to ensure the correct order of events written to the target cluster.

{{% alert title="Tip" color="info" %}}
You can change the batch size by setting the `replicator.reader.pageSize` setting. The maximum value is `4096`, which is also the default value. If you have large events, we recommend changing this setting. For example, you can set it to `1024`.
{{% /alert %}}

If you don't care much about events order in `$all`, you can configure Replicator to use concurrent writers, which will increase performance. The tool uses concurrent writers with a configurable concurrency limit. Writers are partitioned by stream name. This guarantees that events in individual streams will be in the same order as in the source cluster, but the order of `$all` will be slightly off.

To enable concurrent writers, you need to change the `replicator.sink.partitionCount` setting. The default value is `1`, so all the writes are sequential. Do not set this setting to a very high value, as it might lead to thread starvation, or the target database overload. For example, using six to ten partitions is reasonable for a `C4` Event Store Cloud managed database, but higher value might cause degraded performance.
