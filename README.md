# Hardhat

**Lazy about work, paranoid about damage.**

A minimal-code skill for AI coding agents. It runs a laziness ladder — does
this need to exist, is it already here, stdlib, native, installed dependency,
one line — and gates that ladder on the question most minimal-code rules leave
out: *can this be taken back?*

## Why this exists

Minimal-code rules treat every decision as equally cheap to undo. They aren't.
`<input type="date">` is a one-line reversal. A denormalized column shipped to
40 million rows is a migration, a backfill, and a client you don't control.

Hardhat adds three gates, and pays for them by being smaller.

### 1. Gate 0 — reversibility

Runs before the ladder, every time. One question: *if this is wrong in six
months, is the fix a diff, or a migration plus callers you don't own?* On
persisted data shape, published contracts, auth, money, destructive operations
and retention surfaces, the lazy default is **off** — hard stop, and `ultra`
doesn't override it.

This is nearly free in code. `amount_minor INTEGER` with a stated rounding rule
is the same line count as `amount FLOAT`. Gate 0 is an attention tax, not a
code tax.

### 2. Deletion gate

"Deletion over addition" is right, and it is also the change most likely to
cause an outage. Added code fails loudly in the path you just touched; removed
code fails quietly somewhere you never looked, days later, with a stack trace
pointing at the caller instead of your diff.

So removal gets **more** proof than addition, not less — and `grep` is not
proof. Reflection, string-keyed registries, framework autoloading,
class-paths in config, schedulers, serialized blobs, out-of-repo callers and
codegen all reach a symbol without its name appearing at a call site.

### 3. Ship check — correct under repetition

The minimum that works once is routinely wrong called twice. Two questions
before shipping state-touching code: what happens on a retry, and on two
concurrent callers? This class isn't caught by validation or error-handling
rules — the code reads as complete and fails silently under load.

Includes the trap sitting inside the canonical lazy answer: `@lru_cache` on an
`async def` caches the coroutine, not the result.

## What the gates do to the code

Six worked cases. Line counts are non-blank lines of the two implementations,
counted mechanically — not benchmark results. Read them as *what the rule
produces*, not *what it scores*.

| Case | Without | With | Δ |
|---|--:|--:|--:|
| Counter increment, retry-safe | 7 | 3 | **−57%** |
| Create-if-missing, race-safe | 8 | 3 | **−62%** |
| Atomic file write | 14 | 5 | **−64%** |
| Money field (Gate 0) | 3 | 3 | **±0%** |
| Extension point for one processor | 15 | 2 | **−87%** |
| Response cache | 19 | 2 | **−89%** |
| **Total** | **66** | **18** | **−73%** |

The two rows that carry the argument:

**Money field, ±0%.** Gate 0 fires here and changes `amount FLOAT` to
`amount_minor INTEGER` with a rounding rule. Identical line count, and the
float version is a reconciliation project across 200k orders. This is the
whole claim about Gate 0 being affordable, in one row.

**Counter increment, −57%.** The safe version is the *shorter* one —
`UPDATE stats SET views=views+1` instead of select, add, write back. This holds
across the concurrency cases: a unique constraint beats a check-then-act, one
atomic `UPDATE` beats a read-modify-write, a rename beats recovery logic.
Correctness under repetition is mostly not extra code; it's putting the
operation where atomicity already lives — the database, the filesystem — rather
than rebuilding it in application logic.

That's why these gates don't fight the ladder. In five of six cases the
paranoid answer is also the lazy one.

## Not yet benchmarked

The table above is worked cases, not measurement. There is no agentic
benchmark behind this project yet, and no percentage in this README should be
read as one.

To produce real numbers, the harness needs: a fixed repository, ~12 tickets
spanning schema work / deletion / concurrency / plain features, headless agent
runs with and without the skill, n≥4 per condition for variance, and reporting
on LOC, tokens, cost, wall time, and a pass/fail safety check per ticket. Until
that exists and is published, this section stays.

## Cost per turn

An always-on skill is injected context — you pay for it on every turn it
loads. Hardhat's answer is structural: a small always-on core, with depth that
loads only when the situation is live.

| Part | Size | Loaded |
|---|--:|---|
| `SKILL.md` | 94 lines / ~755 words | Every coding turn |
| `references/reversibility.md` | 85 lines | Gate 0 fired, or surface unclear |
| `references/deletion.md` | 59 lines | About to remove code |
| `references/concurrency.md` | 88 lines | State, cache, async, retries, money |
| `references/examples.md` | 84 lines | Calibrating a rung |

316 lines of protocol you don't pay for on a turn that doesn't need it.

The description is also deliberately narrow — it excludes prose, research and
analysis, so the skill doesn't load on turns where it can't help. On a mixed
workload that matters more than the file size does.

## Structure

```
hardhat/
├── SKILL.md                    # always-on core
├── LICENSE
└── references/
    ├── reversibility.md
    ├── deletion.md
    ├── concurrency.md
    └── examples.md
```

## Levels

| Level | Behaviour |
|---|---|
| `lite` | Build what's asked, name the smaller option in one line. |
| `full` | Ladder enforced. Default. |
| `ultra` | YAGNI extremist, deletion-first. **Gate 0 still hard-stops it**, and the deletion proof bar does not drop. |

`ultra` means look harder for things to remove, never remove on weaker
evidence. A confident wrong deletion is the most expensive output this skill
can produce.

## What it will not do

Gate 0 turns off the lazy default; it does not authorise over-engineering. No
abstraction with one implementation, no plugin system for one plugin, no
speculative config. The durable version is the smallest shape that survives
being wrong — usually one or two deliberate decisions, not a framework.

## Licence

MIT. See `LICENSE`.
