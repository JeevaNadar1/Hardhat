# Hardhat

**Lazy about work, paranoid about damage.**

A minimal-code skill for AI coding agents. It runs the same laziness ladder you
already know — does this need to exist, is it already here, stdlib, native,
installed dependency, one line — and gates it on the question that ladder
leaves out: *can this be taken back?*

## Why this exists

### 1. Gate 0 — reversibility

Runs before the ladder, every time. One question: *if this is wrong in six
months, is the fix a diff, or a migration plus callers you don't own?* On
persisted data shape, published contracts, auth, money, destructive operations
and retention surfaces, the lazy default is **off** — hard stop, and `ultra`
doesn't override it.

This costs almost nothing in code. `amount_minor INTEGER` with a stated
rounding rule is the same line count as `amount FLOAT`. Gate 0 is an attention
tax, not a code tax.

### 2. Deletion gate

"Deletion over addition" is right and is also the rung most likely to cause an
outage. Added code fails loudly in the path you touched; removed code fails
quietly somewhere you never looked. Hardhat requires *more* proof for removal
than for addition, and treats `grep` as insufficient — reflection, string-keyed
registries, cron, feature flags, serialized blobs and out-of-repo callers don't
appear in it.

### 3. Ship check — correct under repetition

The minimum that works once is routinely wrong called twice. Two questions
before shipping state-touching code: what happens on a retry, and on two
concurrent callers? This class of bug isn't caught by validation or
error-handling rules — the code reads as complete and fails silently under
load. Includes the trap sitting inside the canonical lazy answer: `@lru_cache`
on an `async def` caches the coroutine, not the result.

Notably, the correct fix here is almost always *shorter* than the broken one —
a unique constraint instead of a check-then-act, one atomic `UPDATE` instead of
a read-modify-write. Safety and laziness point the same direction.

## Cheaper per turn

An always-on skill is injected context: you pay its tokens on every turn, and
its own hooks may inject it into subagents too. So more coverage normally means
higher cost. Hardhat gets both by structure rather than by rules:

| | ponytail v4.9 | hardhat | Δ |
|---|--:|--:|--:|
| Core file, words | 1,069 | 755 | **−29%** |
| Core file, characters | 6,616 | 5,015 | **−24%** |
| Core file, lines | 120 | 94 | −22% |

Three mechanisms:

1. **Progressive disclosure.** The core is the always-on part. Classification tables, the deletion protocol, the concurrency catalogue and the worked examples live in `references/` and load only when the situation is live. Depth you don't pay for on a turn that doesn't need it.
2. **Compression.** The source had genuine redundancy — the comprehension warning appears twice, the persona is restated several ways. Same behaviour, fewer tokens.
3. **Narrower trigger.** The description excludes prose, research and analysis, so the skill doesn't load on turns where it can't help. On a mixed workload this is the largest saving of the three.

Output-side savings are unchanged: the ladder is intact, so the code it
produces is the same size. The gates only fire on the small subset of tasks
where minimal was the wrong default, and on those the extra output is a line or
two of stated reasoning.

## Structure

```
hardhat/
├── SKILL.md                    # always-on core, 94 lines
└── references/
    ├── reversibility.md        # Gate 0 fired, or the surface is unclear
    ├── deletion.md             # about to remove code
    ├── concurrency.md          # state, cache, async, retries, money
    └── examples.md             # calibrating a rung against a real case
```

## Levels

| Level | Behaviour |
|---|---|
| `lite` | Build what's asked, name the smaller option in one line. |
| `full` | Ladder enforced. Default. |
| `ultra` | YAGNI extremist, deletion-first. **Gate 0 still hard-stops it**, and the deletion proof bar does not drop. |

`ultra` means look harder for things to remove, never remove on weaker
evidence.

## What it will not do

Gate 0 turns off the lazy default; it does not authorise over-engineering. No
abstraction with one implementation, no plugin system for one plugin, no
speculative config. The durable version is the smallest shape that survives
being wrong — usually one or two deliberate decisions, not a framework.

## Credit

The ladder, the intensity levels and the debt-comment idea derive from
[ponytail](https://github.com/DietrichGebert/ponytail) by Dietrich Gebert, used
under the MIT licence. The reversibility, deletion and repetition gates, the
compression and the reference structure are additions.

MIT.
