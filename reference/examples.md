# Calibration

Worked cases. The point of each is which gate or rung fires, and why the
neighbouring answer is wrong.

## Gate 0 fires

**"Add a discount field to orders."**
Persisted shape → irreversible. Lazy answer is `discount FLOAT`. Durable answer
is `discount_minor INTEGER` plus a stated rounding rule, because a float
rounding error across 200k orders is a reconciliation project, not a patch.
Same line count. Gate 0 cost nothing here except the decision.

**"Just return the user's name from the endpoint."**
Published to callers → irreversible. `return name` is one line and traps you:
adding email next is a breaking change for every client parsing a bare string.
`return {"name": name}` is also one line and additive forever.

**"Clean up the old sessions table."**
Terminal. Stop and confirm before acting — no amount of laziness justifies
guessing here.

**"Cache these API responses."**
Reversible → ladder applies. `@lru_cache(maxsize=1000)`, skipped the cache
class. Then the ship check: is the function async? If yes, `lru_cache` is
wrong. Rung 3 gave the right *shape*, the ship check caught the bug in it.

## Rung 4 and the support floor

**"Build a date picker."**
Native `<input type="date">` is the right call for an internal admin tool on
evergreen browsers: 400 lines become one. It is the wrong call for a booking
flow needing a range, blackout dates, or a design system match — the native
control is near-unstyleable and behaves differently per browser and locale.
Rung 4 is a rung, not a law. Name the target and check it before taking it.

**"Validate this email."**
Rung 4/5 territory: the platform's own validation, or the validator already
installed. Never a hand-rolled regex. But do not reach for a new dependency
either — one of these two already exists.

## Deletion gate

**"This handler looks unused, remove it."**
Name search comes back clean. It's registered in a string-keyed route table,
so `grep` for `def handler(` finds nothing and the route still resolves.
Exported to callers you don't own → not a deletion, a deprecation.

**"Delete this commented-out block."**
Delete on sight. No protocol.

## The ladder working normally

**"Group these records by category."**
Rung 3: `itertools.groupby` after a sort, or `collections.defaultdict`. Not a
`GroupingStrategy` class. One line, no check needed — trivial.

**"Add retry logic to this HTTP call."**
Rung 5: the retry adapter in the HTTP client already installed. Then the ship
check fires, because retries make idempotency mandatory — if the endpoint
creates something, the retry needs a key. This is the pairing the ladder alone
misses: rung 5 gives you retries in one line and silently creates a duplicate-
write problem.

**"We need a plugin system so we can add processors later."**
Rung 1. One processor exists. Skip it, say so in one line, write the function.
Later can add the second function; the third one earns the abstraction.

## Level differences on one request

**"Add a cache."**

- **lite** — "Added. `functools.lru_cache` covers this in one line if you'd rather not own a cache class."
- **full** — "`@lru_cache(maxsize=1000)` on the fetch. Skipped the cache class; add when it measurably falls short."
- **ultra** — "No cache until a profiler asks for one. When it does: `@lru_cache`. A hand-rolled TTL class is a bug farm with a hit rate."

All three change on Gate 0 if the "cache" is a persisted table rather than an
in-process dict — then it's a schema decision and the small answer is off.

## The failure this skill exists to prevent

A minimal diff, in the right file, that reads as complete, passes review, and
cannot be undone. Small and wrong is worse than large and wrong, because
nobody looks at it twice.
