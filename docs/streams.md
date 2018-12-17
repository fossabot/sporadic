# sporadic.streams

Reactive streams on top of Promises.

This submodule provides an implementation glitch-free (i.e, no lost events) and
leak-free (i.e, garbage collection is possible) for reactive programming. The
minimal abstraction is called _stream_, used to associate many producers to many
consumers.

Although not real-time (a.k.a, the synchronous / instantaneous axiom /
assumption found on Synchronous Imperative languages, such as Lucid), this
`sporadic.streams` submodule offers asynchronous reactivity without glitches.

The API revolves around an `async` / `await` style, and thus, a promise / future
concurrency model. Consumers read deterministically published values (the same
stream point yields the same event), while producers may non-deterministically
race / dispute the order of events arrival. Streams are closed for further writes
and reads by _breaking_ / _rejecting_ the stream in a promise-style through the
close operation.

Garbage collection comes for free by implementing a linked list of streamed /
promisified `next` nodes. That is, if the current node is not referenced anymore
and the next node is resolved (by sending some value into the current one), so
the current node becomes prone to garbage collection. So, in short terms, pay
attention on your stream point references, and discard them whenever you can and
whenever you don't need them anymore.

## API Usage

To load this submodule:

```javascript
const sporadicStreams = require('sporadic').streams
```

---

Creates a new reactive stream:

```javascript
const stream = await sporadicStreams.open()
```

---

Write on a reactive stream:

```javascript
const next = await sporadicStreams.push(stream, value)
```

Where `next` is the next stream to send values onto. This operation may fail due
a sent close signal. You can reuse many times the `stream` reference instead of
tracking `next`, for example:

```javascript
await sporadicStreams.push(stream, 'Hello, Mike!')
await sporadicStreams.push(stream, 'Are you fine?')
await sporadicStreams.push(stream, 'See you later!')
```

Although it's a discouraged pattern, 'cause it forces you to keep all the stream
point references in-memory, without discarding any of them, a huge risk of
memory leaking for long-running applications. Don't rely on that except for
short running tests. **You have read this warning.** The good solution, so, is
a _threading_ of references (the example below show how it's possible):

```javascript
const stream1 = await sporadicStreams.push(stream0, 'Hello, Mike!')
const stream2 = await sporadicStreams.push(stream1, 'Are you fine?')
const stream3 = await sporadicStreams.push(stream2, 'See you later!')
// ...
```

Through non-determinism on writes, clients can get the next stream point in
unpredictable ways (only if there's more than one writer). The threading above
seems deterministic cause there's no race. It allows garbage collection, only if
further references aren't part of the same scope -- the best solution here, tho,
would be to iterate the generated stream points through a loop, or use them
together with generators+promises (as a task scheduler).

---

To consume / read a value:

```javascript
// may throw reason
const {
  current, next
} = await sporadicStreams.pull(stream)
```

Where `next` is the next stream point to read future pushed values, and
`current` is the current pushed value at this stream point. This operation may
fail due a sent close signal.

---

To destroy an active stream:

```javascript
// throws reason
await sporadicStreams.close(stream, reason)
```

Previous clients might not be up-to-date (due late computations), so they will
keep reading values until this `reason` is available, then they will break /
fail with that.

---

To import all operations on the current scope, you can use the following
pattern:

```javascript
const { open, push, close, pull } = require('sporadic').streams
```


## References:

- [Lucid Programming Language](https://en.wikipedia.org/wiki/Lucid_(programming_language)