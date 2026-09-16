# Reversibility

The gate the laziness ladder was missing. Minimal code is cheap to undo in
application logic and ruinous to undo anywhere state, contracts, or money
persist. This file decides which you're in.

## The test

> If this decision is wrong in six months, what does the fix cost?

| Answer | Class | Behaviour |
|---|---|---|
| A diff, merged and deployed | **Reversible** | Climb the ladder. |
| A diff plus a data migration | **Sticky** | Ladder applies to the *code*, not the *shape*. Get the shape right. |
| A migration, a backfill, and coordinating callers you don't control | **Irreversible** | **Lazy default off.** Build the durable version. |
| Cannot be undone at all (data destroyed, keys rotated, invoices sent) | **Terminal** | Stop. Confirm with the user before acting. |

Unclassifiable → treat as irreversible. The cost of over-building a reversible
thing is one afternoon. The cost of under-building an irreversible one is a
quarter.

## The six surfaces

**1. Persisted data shape.** Table and column semantics, nullability, types,
enum values, denormalization, index-implied access patterns, document shape in
a schemaless store. Once rows exist, the shape is a contract with the past.
*Durable version:* name things for what they are, not what today's feature
calls them; nullable until proven required; store the raw value and derive the
convenient one; never overload a column with two meanings.

**2. Anything published to a caller you don't own.** Public or partner API
routes and response bodies, event and message payloads, webhook shapes, SDK
signatures, CLI flags and output, file formats, URL structure. The moment a
third party parses it, you can't change it on your schedule.
*Durable version:* additive-only changes; version at the boundary; return
objects rather than bare scalars so fields can be added; never reuse a field
name for a new meaning.

**3. Auth and identity.** Token format and claims, session model, permission
and role shape, tenancy boundary, password hashing choice, key derivation. A
permission model is a data migration *and* a security review to change.
*Durable version:* explicit tenancy from day one, even with one tenant; deny by
default; permissions as data, not as branches in code.

**4. Money.** Currency units and precision, rounding rule and direction,
idempotency keys, transaction boundaries, ledger append vs mutate, tax and fee
ordering. Wrong rounding is not a bug you patch; it's a reconciliation project.
*Durable version:* integer minor units, never floats; rounding rule stated once
and centrally; every externally-triggered write idempotent; ledger entries
append-only.

**5. Destructive operations.** Deletes, overwrites, truncation, dropping a
column, cache invalidation that loses the only copy, in-place file rewrites.
*Durable version:* soft-delete or tombstone before hard-delete; write to a new
location and swap; make the destructive path require an explicit flag.

**6. Retention and audit.** Anything with a legal hold, an audit trail, or a
regulatory retention window. Minimal logging here is a compliance finding.

## What "durable version" does not mean

Gate 0 turns off the *lazy default*. It does not authorise over-engineering.
You still don't get: an abstraction with one implementation, a plugin system,
speculative fields, or a config surface nobody asked for. The durable version
is *the smallest shape that survives being wrong* — usually one or two
decisions made deliberately, not a framework.

Concretely: choosing `amount_minor INTEGER` plus a stated rounding rule over
`amount FLOAT` costs zero extra lines. That's the whole move. Gate 0 is mostly
free; it's an attention tax, not a code tax.

## Sticky, the middle case

Most schema work is sticky rather than irreversible: the shape is expensive to
change but you own every caller. Here the ladder still applies to the
implementation — no repository pattern, no ORM abstraction layer, no service
class — while the *shape* gets deliberate thought. Lazy code over a considered
schema is the correct combination and the one this gate is designed to produce.

## Saying it

One line, no essay, then build:

> "Schema change — not taking the lazy default here. `amount_minor INTEGER`
> with half-up rounding at the API edge. Rest of it is the two-line handler."
