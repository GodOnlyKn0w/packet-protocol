# Packet v3

Packet is the transport layer for cognitive projection between projects.

[tasktree](https://github.com/GodOnlyKn0w/strand-protocol) stores cognition
within a project. Packet delivers constraints across project boundaries.
Separate repos, separate roles.

## Quick Start

A packet is one line of JSONL:

```jsonl
{"schema":"packet-v3","id":"pkt_001","semantic_hash":"sha256...","kind":"constrain","source_scope":"local:project-alpha","target_scope":"remote:project-beta","created_at":"2026-05-02T10:00:00Z","body":"Do not delete untracked files. Use .gitignore instead.","trace":"strand:example-001"}
```

Send it by copying to the target project's `.packets/inbox/`.
Consume it by appending to a strand:

```
[decision] imported constrain: Do not delete untracked files.
source_packet: pkt_001
```

## Four Kinds

| Kind | Direction | Binding |
|------|-----------|:-------:|
| constrain | downstream | ✅ only binding kind |
| status | upstream | read-only |
| incident | bidirectional | attention trigger |
| next_open | downstream | direction hint |

## Relation to tasktree

```
tasktree    stores cognition within a project (journal, strands)
Packet      delivers constraints across project boundaries
```

Packet does not replace tasktree. A consumed constrain lands in a
tasktree strand as `[decision] imported constrain`, not as a native
local decision. The distinction is intentional.

## Spec

See [PROTOCOL.md](./PROTOCOL.md) for the full specification.

## Examples

See [examples/](./examples/) for one packet per kind.

## License

MIT
