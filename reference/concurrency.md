# Correct under repetition

The minimum that works is usually verified once, sequentially, by hand. Real
callers retry, and there are several of them at the same time. This class of
bug is not caught by the never-simplify-away list because it doesn't look like
missing validation or missing error handling — the code reads as complete. It
fails silently, under load, in production, and the diff looks fine in review.

Two questions, before shipping anything that touches state, cache, async, or
an external trigger:

1. **What happens on a retry?** (client retry, queue redelivery, webhook resend, user double-click, deploy replay)
2. **What happens with two concurrent callers?**

If the answer is "corruption", "double charge", "lost update", or "I don't
know", it isn't done.

## The recurring failures

**Read-modify-write.** `x = get(); x.count += 1; save(x)` loses updates under
concurrency. The lazy *and* correct fix is almost always shorter than the code
that's wrong: `UPDATE t SET count = count + 1 WHERE id = ?`. Push the mutation
into the database rather than reconstructing it in application code.

**Check-then-act.** `if not exists(k): create(k)` races. Use the unique
constraint you already have and catch the violation, or `INSERT ... ON
CONFLICT DO NOTHING`. Again: fewer lines than the broken version.

**Non-idempotent handlers.** Any endpoint or consumer that creates, charges,
sends, or increments will be invoked twice. Idempotency key or natural unique
key — and note this is a Gate 0 surface, because retrofitting idempotency onto
a live endpoint means reconciling the duplicates it already created.

**Caching a coroutine.** `@lru_cache` on an `async def` caches the coroutine
object, not the result; the second caller awaits an already-consumed
coroutine. This is the exact failure hiding in the "just use `lru_cache`"
answer, and it's why the ladder's rung-3 reflex needs this check downstream of
it. Cache the awaited value, or use an async-aware cache.

**Unbounded cache as a leak.** `@lru_cache` with no `maxsize` on a
user-keyed function is a memory leak with a hit rate. Also: it pins `self`
alive on methods.

**Fire-and-forget async.** A task created without a reference is
garbage-collectable mid-flight and its exception is swallowed. Hold the
reference or await it.

**Mutable default argument / module-level mutable state.** Shared across every
call and every request in the process. Classic minimal-code trap.

**Non-atomic file writes.** Writing in place means a crash leaves a truncated
file. Write to a temp file in the same directory and rename — atomic on POSIX,
and shorter than any recovery logic.

**Time and ordering.** `now()` read twice in one operation gives two answers.
Wall-clock for elapsed time goes backwards on NTP correction; use a monotonic
clock. Events do not arrive in order.

**Transaction scope.** A transaction that spans a network call holds a lock for
the duration of someone else's latency. Do the I/O outside, the write inside.

## The bias worth internalising

For this failure class, the correct solution is usually **shorter** than the
incorrect one — a unique constraint instead of a check, one atomic `UPDATE`
instead of a read-modify-write, a rename instead of recovery code. Concurrency
safety is mostly not extra code; it's putting the operation where atomicity
already lives (the database, the filesystem, the type system) rather than
rebuilding it in application logic.

That makes this check fully compatible with laziness. It is not a licence to
add a lock, a queue, or a distributed coordinator. If your answer to a race is
a new dependency, you've climbed the wrong rung — look for the constraint or
atomic primitive already available first.

## The check to leave behind

Rule 5 wants one runnable check for non-trivial logic. For this class, the
check that earns its place is the one that calls the thing twice:

```python
def demo():
    reset()
    handle(request_id="r1"); handle(request_id="r1")   # retry
    assert count() == 1, "handler is not idempotent"
```

Two lines, catches the bug the reviewer won't.
