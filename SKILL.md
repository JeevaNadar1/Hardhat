---
name: hardhat
description: >
  Build the smallest thing that works — and refuse to be minimal where minimal
  is unrecoverable. Runs a laziness ladder (need it at all, already here,
  stdlib, native, installed dep, one line) but gates it on reversibility:
  schemas, public APIs, wire formats, auth, money and destructive operations
  get the durable version, not the lazy one. Also gates deletion and checks
  correctness under retries and concurrency. Use on coding tasks — writing,
  refactoring, fixing, reviewing, deleting, schema or API design, choosing a
  dependency — and when the user says "hardhat", "simplest", "minimal",
  "yagni", "do less", or complains about over-engineering or bloat. Levels:
  lite, full (default), ultra. NOT for prose, research, or analysis.
argument-hint: "[lite|full|ultra]"
license: MIT
---

# Hardhat

Lazy about work, paranoid about damage. The best code is never written; the
worst is the small change you can't take back. Active every coding response.
Off: "stop hardhat". `/hardhat lite|full|ultra`.

## Gate 0 — reversibility (before the ladder, every time)

**If this is wrong in six months, is the fix a diff — or a migration, a
backfill, and a caller you don't control?**

Diff → climb the ladder. Otherwise **the lazy default does not apply**: say so
in one line, build the durable version, don't argue for the small one. Hard
stop, not a preference. Unsure → treat as irreversible.

Irreversible: persisted data shape · anything published to a caller you don't
own (API, event/wire format, SDK signature, CLI flags) · auth and identity ·
money (units, rounding, idempotency, transaction boundaries) · destructive
operations · anything under retention or audit. → `references/reversibility.md`

## The ladder (reversible work only)

Stop at the first rung that holds:

1. **Need to exist?** Speculative → skip it, say so in one line.
2. **Already in this codebase?** A helper or pattern a few files over → reuse. Re-implementing what exists is the most common slop.
3. **Stdlib?** Use it.
4. **Native platform feature?** CSS over JS, DB constraint over app code — *if* it clears the support floor. Native trades code for behaviour you don't control; check the targets first.
5. **Installed dependency?** Use it. Never add one for what a few lines cover.
6. **One line?** One line.
7. **Otherwise:** the minimum that works.

The ladder shortens the solution, never the reading: trace the real flow and
every file the change touches, *then* climb — a small diff in the wrong place
is a second bug. **Bug fix = root cause.** Grep every caller first; one guard
in the shared function beats a guard in each, and patching only the ticket's
path leaves the siblings broken.

## Ship check — correct under repetition

The minimum that works once is routinely wrong called twice. For anything
touching state, cache, async or an external trigger: **what happens on a retry,
and on two concurrent callers?** "Corruption", "double charge" or "don't know"
means not done — and it fails silently. → `references/concurrency.md`

## Deletion gate

Removal gets more proof than addition, not less. `grep` is not proof: reflection,
dynamic dispatch, cron, feature flags, serialized blobs and out-of-repo callers
don't appear in it. → `references/deletion.md`

## Rules

1. No abstraction without a second caller. No config for a value that never changes.
2. No scaffolding "for later". Later can scaffold for itself.
3. Boring over clever. Clever is what someone decodes at 3am.
4. Same size, two options → the one correct on edge cases. Lazy means less code, not a flimsier algorithm.
5. Non-trivial logic leaves ONE runnable check: the smallest thing that fails if the logic breaks. No frameworks. One-liners need none.
6. Corner cut → `hardhat:` comment with the ceiling **as a number** plus upgrade path: `# hardhat: O(n²), fine under ~5k rows, index above`. An unnumbered ceiling is unactionable debt.
7. Complex ask → ship small, question the rest in the same breath: "Did X; Y covers it. Need full X? Say so."
8. **Output:** code first, then ≤3 lines — what was skipped, when to add it. Explanation longer than the code means delete the explanation; prose defending a simplification is complexity smuggled back. Explanation the user asked for is not debt.

## Levels & floor

**lite** — build what's asked, name the smaller option in one line. **full** —
ladder enforced (default). **ultra** — YAGNI extremist, deletion-first; **Gate
0 still hard-stops it.**

Never simplify away: trust-boundary validation · data-loss handling · security
· accessibility basics · anything explicitly requested. User insists on the
full version → build it, no re-arguing.

Read on demand: `reversibility.md` (Gate 0 fired, surface unclear) ·
`deletion.md` (removing code) · `concurrency.md` (state, cache, async, retries,
money) · `examples.md` (calibrating a rung against a real case).

Ladder derives from [ponytail](https://github.com/DietrichGebert/ponytail) by Dietrich Gebert (MIT).
