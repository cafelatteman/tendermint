# ADR 024: Pubsub

Author: Anton Kaliaev (@melekes)

## Changelog

02-10-2018: Initial draft

## Context

Since the initial version of the pubsub, there's been a number of issues
raised: #951, #1879, #1880. Some of them are high-level issues questioning the
core design choices made by us. Others are minor and mostly about the interface
of `Subscribe()` / `Publish()` functions.

### Sync vs Async

Now, when publishing a message to subscribers, we can do it in a goroutine:

```go
for each subscriber {
    out := subscriber.outc
    go func() {
        out <- msg
    }
}
```
or
```go
for each subscriber {
    go subscriber.callbackFn()
}
```

This gives us greater performance and allow us to avoid "slow client problem"
(when other subscribers have to wait for a slow subscriber). A pool of
goroutines can be used to avoid uncontrolled memory growth.

In certain cases, this is what we want. In other cases, when for example we
need strict ordering of events (if event A was published before B, the delivery
order will be A -> B), we can't use goroutines.

There is also a question whenever we should have a non-blocking send:

```go
for each subscriber {
    out := subscriber.outc
    select {
        case out <- msg:
        default:
            log("subscriber %v buffer is full, skipping...")
    }
}
```

This fixes the "slow client problem", but there is no way for a slow client to
know if it had missed a message.

### Channels vs Callbacks

TODO

### Why `Subscribe()` accepts an `out` channel?

TODO


## Decision

## Status

In development

## Consequences

### Positive

### Negative

### Neutral
