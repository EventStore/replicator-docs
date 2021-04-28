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

{{< alert title="Warning:" >}}
JavaScript transforms only work with JSON payloads.
{{< /alert >}}

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
- `data` - event payload as an object
- `metadata` - event metadata as an object

The function must return an object, which contains `stream`, `eventType`, `data` and `metadata` fields. Both `data` and `metadata` must be valid objects, the `metadata` field can be `undefined`. If you haven't changed the event data, you can pass `data` and `metadata` arguments, which the function receives as arguments.

## Logging

You can log from JavaScript code directly to Replicator logs. Use the `log` object with `debug`, `info`, `warn` and `error`. You can use string interpolation as usual, or pass templated strings in [Serilog format](https://github.com/serilog/serilog/wiki/Writing-Log-Events). The first parameter is the template string, plus you can pass up to five additional values, which could be values or objects.

For example:

```javascript
log.info(
    "Transforming event {@Data} of type {Type}", 
    original.data, original.eventType
);
```

Remember that the default log level is `Information`, so debug logs won't be shown. Enable debug-level logging by setting the `REPLICATOR_DEBUG` environment variable to `true`.

## Example

Here is an example of a transformation function, which changes the event data, stream name, and event type:

```js
function transform(original) {
    const newEvent = {
        ...original.data.event,
        Data1: `new${original.data.Data1}`,
        NewProp: `${original.data.Id} - ${original.data.Data2}`
    };
    return {
        stream: `transformed${stream}`,
        eventType: `V2.${eventType}`,
        data: newEvent,
        meta
    }
}
```

If the function returns `undefined`, the event will not be replicated, so the JavaScript transform can also be used as an advanced filter. The same happens if the transform function returns an event, but either the stream name or event type is empty or `undefined`.
