# packet-protocol

Packet is a cross-boundary cognitive projection protocol. It defines how to
compress, deliver, and trace constraints between projects that each maintain
their own tasktree journal.

Packet is the transport layer. [tasktree](https://github.com/GodOnlyKn0w/strand-protocol)
is the storage layer. Separate repos, separate roles.

## Quick Start

1. Read [PROTOCOL.md](PROTOCOL.md) for the v3 specification
2. See [examples/](examples/) for minimal JSONL packets of each kind

## Repository Relationship

```
strand-protocol     tasktree SPEC (journal format, checkpoint protocol)
packet-protocol     Packet SPEC (cross-boundary delivery format)
strand-seed         Public reference implementation (bundled tasktree binary)
```

## License

MIT — see [LICENSE](LICENSE)
