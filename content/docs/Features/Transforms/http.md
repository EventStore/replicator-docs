---
title: "HTTP transformation"
linkTitle: "HTTP"
weight: 2
date: 2021-04-08
description: >
    Transform and filter events using out-of-process HTTP server
---

## Introduction

An HTTP transformation can be used for more complex changes in event schema or data, which is done in an external process. It allows you to use any language and stack, and also call external infrastructure to enrich events with additional data.

When configured accordingly, Replicator will call an external HTTP endpoint for each event it processes, and expect a converted event back. As event data is delivered as-is (as bytes) to the transformer, there's no limitation of the event content type and serialisation format.

{{% alert title="Note" color="primary" %}}
Right now, events are sent to the transformer one by one. It guarantees the same order of events in the sink store, but it may be slow. We plan to enable event batching to speed up external transformations.
{{% /alert %}}

## Guidelines

Before using the HTTP transformation, you need to build and deploy the transformation function, which is accessible using an HTTP(S) endpoint. The endpoint is built and controlled by you. It must return a response with a transformed event with `200` status code, or `204` status code with no payload. When the Replicator receives a `204` back, it will not propagate the event, so it also works as an advanced filter.

The transformation configuration has two parameters:
- `replicator.transform.type` - should be `http` for HTTP transform
- `replicator.transform.config` - the HTTP(S) endpoint URL

For example:

```yaml
replicator:
  transform:
    type: http
    config: http://transform-func.myapp.svc.cluster.local
```

Replicator doesn't support any authentication, so the endpoint must be open and accessible. You can host it at the same place as Replicator itself to avoid the network latency, or elsewhere. For example, your transformation service can be running in the same Kubernetes cluster, so you can provide the internal DNS name to its service. Alternatively, you can use a serverless function.

Replicator will call your endpoint using a `POST` request with JSON body. The request and response formats are the same:

```json
{
    "eventType": "string",
    "streamName": "string",
    "payload": "string"
}
```

The `payload` field contains the binary event data payload as UTF8 string. If you use JSON in your events, you'll get a JSON string, which you can then deserialize.

You can change any field value when returning the result. Therefore, you have full control to change the event type, stream name and the payload. If your endpoint returns `204`, the event will be ignored, and we won't replicate it.

## Example

Here is an example of a serverless function in GCP, which transforms each part of the original event:

```csharp
using System.Text.Json;
using Google.Cloud.Functions.Framework;
using Microsoft.AspNetCore.Http;
using System.Threading.Tasks;

namespace HelloWorld {
    public class Function : IHttpFunction {
        public async Task HandleAsync(HttpContext context) {
            var original = await JsonSerializer
                .DeserializeAsync<HttpEvent>(context.Request.Body);

            var payload = JsonSerializer
                .Deserialize<TestEvent>(original.Payload);
            payload.Data1 = $"Manipulated {payload.Data1}";

            var proposed = new HttpEvent {
                StreamName = $"new-{original.StreamName}",
                EventType  = $"V2.{original.EventType}",
                Payload    = JsonSerializer.Serialize(payload)
            };

            await context.Response
                .WriteAsync(JsonSerializer.Serialize(proposed));
        }

        class HttpEvent {
            public string EventType  { get; set; }
            public string StreamName { get; set; }
            public string Payload    { get; set; }
        }

        class TestEvent {
            public string Id    { get; set; }
            public string Data1 { get; set; }
            public string Data2 { get; set; }
            public string Data3 { get; set; }
        }
    }
}
```

The `TestEvent` is the original event contract, which is kept the same. However, you are free to change the event schema too, if necessary.
