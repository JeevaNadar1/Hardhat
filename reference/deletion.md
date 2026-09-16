# Deletion

"Deletion over addition" is right, and it is the rung most likely to cause an
outage. Added code that's wrong usually fails loudly in the path you just
touched. Removed code that was load-bearing fails somewhere you never looked,
often days later, and the stack trace points at the caller rather than at your
diff.

So: **removal gets more proof than addition, not less.** This is the asymmetry
the original ladder had backwards.

## Why grep is not proof

A symbol can be reached without its name appearing at any call site:

| Mechanism | Looks like |
|---|---|
| Reflection / dynamic dispatch | `getattr(mod, name)`, `cls.__subclasses__()`, `importlib`, Java reflection, `eval` |
| String-keyed registries | route tables, DI containers, plugin maps, event-name dispatch, Celery/queue task names |
| Framework convention | Django/Rails/Spring autoloading, migration files, admin registration, serializer field lookup, template variable resolution |
| Config and data | a class path in a YAML/env value, a handler name in a DB row, a feature flag naming a strategy |
| Schedulers | cron, systemd timers, CI workflows, Airflow DAGs, cloud scheduler jobs |
| Serialized state | pickles, cached objects, queued job payloads, session blobs referencing a class that must still exist to deserialise |
| Out-of-repo callers | another service, a mobile client on an old version, a partner integration, someone's notebook |
| Build-time | codegen output, macros, decorators registering side-effectfully at import |

Any one of these makes a clean `grep` result meaningless.

## The protocol

Before removing anything non-trivial:

1. **Search by name, not just call syntax.** The bare string, not `foo(`. Include config, templates, migrations, workflow files, and any `.json`/`.yaml`/`.toml`.
2. **Check the dynamic mechanisms above** that the codebase actually uses. Identify which apply here — most repos use two or three.
3. **Ask whether anything outside the repo can reach it.** A route, an event name, an exported symbol, a public class. If yes, this is Gate 0 territory, not a deletion.
4. **Check the serialization surface.** Is this class or field present in anything already persisted or queued? Removing it breaks deserialisation of existing data.
5. **Prefer proof over inference where it's cheap.** Runtime evidence beats reading: a log line or metric on the suspect path, deployed for one full traffic cycle, settles it. For anything reachable in production, this is the default, not the thorough option.
6. **Stage it when the blast radius is unclear:** deprecate → warn → remove, or remove behind a flag. Two small diffs beat one confident one.

## The tiers

| Situation | Proof required |
|---|---|
| Local variable, private helper with one caller in the same file, dead branch you can read end to end | Read it. Delete. |
| Module-private function, internal class, unused import | Name search + the dynamic mechanisms this repo uses. |
| Anything exported, registered, routed, scheduled, or serialised | Runtime evidence, or staged removal. |
| Anything a caller outside the repo could reach | Not a deletion. Deprecation with a timeline. |

## Under `ultra`

`ultra` is deletion-first and that stays. What it does not get is a lower
proof bar. Ultra means *look harder for things to remove*, never *remove on
weaker evidence*. A confident wrong deletion is the single most expensive
output this skill can produce.

## Commented-out code

Delete on sight, no protocol. Version control is the archive. This is the one
deletion that needs no proof.
