---
title: "JavaScript transformation"
linkTitle: "JavaScript"
weight: 1
date: 2021-03-05
description: >
    Transform and filter events using in-proc JavaScript function
---

## Introduction

When you need to perform simple changes in the event schema, change the stream name or event type based on the existing event details and data, you can use a JavaScript transform.

For this transform, you need to supply a code snippet, written in JavaScript, which does the transformation. The code snippet must be placed in a separate file, and it cannot have any external dependencies. There's no limitation on how complex the code is. Replicator uses the V8 engine to execute JavaScript code. Therefore, this transform normally doesn't create a lot of overhead for the replication.

## Guidelines

You can configure Replicator to use a JavaScript transformation function using the following parameters:

- `replicator.transform.type` - must be set to `js`
- `replicator.transform.config` - name of the file, which contains the transformation function

For example:

```yaml
replicator:
  transform:
    type: js
    config: ./transform.js
```

The function itself must be named `transform`. Replicator will call it with the following arguments:

- `stream` - original stream name
- `eventType` - original event type
- `data` - event payload as UTF8 string
- `metadata` - event metadata as UTF8 string

If your events are serialized in JSON, you can use `JSON.parse` to deserialize the payload or metadata.

The function must return an object, which contains `stream`, `eventType`, `data` and `metadata` fields. Both `data` and `metadata` must be strings. For JSON payloads, you can use `JSON.stringify` to serialise your objects. If you haven't changed the event data, you can pass `data` and `metadata` arguments, which the function receives as arguments.

## Example

Here is an example of a transformation function, which changes the event data, stream name, and event type:

```js
function transform(stream, eventType, data, meta) {
    const event = JSON.parse(data);
    const newEvent = {
        ...event,
        Data1: `new${event.Data1}`,
        NewProp: `${event.Id} - ${event.Data2}`
    };
    return {
        stream: `transformed${stream}`,
        eventType: `V2.${eventType}`,
        data: JSON.stringify(newEvent),
        meta
    }
}
```

If the function returns `null`, the event will not be replicated, so the JavaScript transform can also be used as an advanced filter.
