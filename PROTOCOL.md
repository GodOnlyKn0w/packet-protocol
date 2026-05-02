# Packet v3 — Cross-Boundary Cognitive Projection Protocol

Packet is the transport layer for cognitive projection between projects.
tasktree stores cognition within a project. Packet delivers constraints
across project boundaries. Separate repos, separate roles.

## Three Primitives

| Primitive | Role |
|-----------|------|
| project | Compress binding information from source context into deliverable form |
| deliver | Transfer across a project boundary |
| trace | Preserve a pointer back to the source reasoning (optional) |

Primitive names are distinct from kind names. constrain / status / incident /
next_open are kinds only.

## Four Kinds

| Kind | Direction | Binding Force |
|------|-----------|:-------------:|
| constrain | downstream | Yes (the only binding kind) |
| status | upstream | No (read-only) |
| incident | bidirectional | No (attention trigger) |
| next_open | downstream | No (direction hint) |

A consumed constrain is written as `[decision] imported constrain: ...
source_packet: <id>`. This distinguishes a locally accepted external
constraint from a native local decision — they are not equivalent.

## Format

One line of JSONL:

```jsonl
{"schema":"packet-v3","id":"pkt_xxx","semantic_hash":"sha256...","kind":"constrain","source_scope":"local:project-alpha","target_scope":"remote:project-beta","created_at":"2026-05-02T10:00:00Z","body":"<message>","trace":"strand:<id>"}
```

### Fields

| Field | Required | Description |
|-------|:--------:|-------------|
| schema | yes | `packet-v3` |
| id | yes | Unique message ID, used for per-message idempotency |
| semantic_hash | yes | `sha256(source_scope + target_scope + kind + canonical_body)`, used for cross-message deduplication |
| kind | yes | `constrain` / `status` / `incident` / `next_open` |
| source_scope | yes | Source scope alias (arbitrary string) |
| target_scope | yes | Target scope alias (arbitrary string) |
| created_at | yes | ISO 8601 |
| body | yes | Message body |
| trace | no | Pointer to source strand id |

### id and semantic_hash Separation

`id` is a per-message unique identifier (may include a nonce or timestamp).
`semantic_hash` is a content fingerprint (excludes nonce). When two agents
independently generate packets with the same constraint, their `id` fields
differ but `semantic_hash` matches. Consumption checks `semantic_hash` first
to avoid duplicate writes.

### canonical_body

`canonical_body = trim(body) + normalize whitespace + normalize line endings`

Text-level normalization only. No semantic normalization — translation,
synonym replacement, and abbreviation expansion are out of scope.

### Scope Aliases

Packet files contain only aliases (e.g. `local:project-alpha`). Real repository
names, paths, or URLs never appear in packet files. The alias-to-real mapping
is maintained locally by each project, outside packet files and outside
version control.

## Storage

```
.packets/outbox/     Pending delivery
.packets/inbox/      Received
.packets/processed/  Consumption markers
  <packet_id>.ack          ← Exact dedup
  <semantic_hash>.seen     ← Semantic dedup
```

## Consumption Rules

```
read inbox/
for each packet:
  if semantic_hash.seen → skip (semantic dedup)
  if id.ack → skip (exact dedup)
  if id in journal → skip (journal is authoritative)
  → append to strand with appropriate prefix:
    constrain → "[decision] imported constrain: ... source_packet: <id>"
    incident  → "[observation] incident: ..."
    next_open → "[open] ..."
    status    → "[observation] status: ..."
  → create id.ack + semantic_hash.seen
```

## Conflict Rules

```
constrain > next_open > incident > status
local decision > remote packet
```

Newer constrains do not automatically override older ones. Conflicts are
recorded (`conflict_id = sha256(packet_a + packet_b)`), not resolved.
Resolution is the responsibility of the consuming operator/agent.

## Design Constraints

- No new binary — write JSONL by hand until friction demands tooling
- One complete packet per line, not a lifecycle event stream
- `.packets/` is independent of `.tasktree/`
- Delivery responsibility is explicitly assigned to the source project's agent
- Git is an optional transport, not a protocol requirement
- Under-delivery is preferable to inbox noise — a missing constraint is easier
  to detect and correct than a flooded inbox
