# go-facter

**A pure-Go (no cgo) reimplementation of Puppet's
[Facter](https://www.puppet.com/docs/puppet/latest/facter.html) system-inventory
engine.**

go-facter discovers structured facts about the host — operating system, kernel,
networking, processors, memory, filesystems, virtualization, uptime, identity and
more — and exposes them through a small, stable Go API designed to be bound onto
Ruby's `Facter` interface (`Facter.value`, `Facter[]`, `Facter.add`,
`Facter.to_hash`).

It is the **engine layer**: a sibling `go-ruby-facter` binds the Ruby Facter API
onto it so Puppet manifests and Ruby code running under `rbgo` can resolve facts.
It is a **standalone, dependency-free** module — it imports only the Go standard
library.

```go
c := facter.New()
name, _ := c.Value("os.name")             // -> Facter.value("os.name")
all := c.ToHash()                          // -> Facter.to_hash
c.AddValue("role", "web")                  // -> Facter.add
c.LoadExternalFacts("/etc/facter/facts.d") // external facts
```

## Why it exists

Puppet's Facter is Ruby with C extensions and shells out heavily. To run Puppet
manifests and Ruby fact code under a pure-Go embedded Ruby (`rbgo`) with **cgo
disabled and a static binary**, the fact engine has to be pure Go too. go-facter
provides that engine, matching Facter's fact **names and schema** so it is a
drop-in.

- **[Why a Go engine](why.md)** — the split between engine and Ruby surface.
- **[Facts & schema](facts.md)** — every fact group and its structure.
- **[Usage & API](api.md)** — the Go API the Ruby layer maps onto, and the CLI.
- **[Custom & external facts](external.md)** — extending the fact set.
- **[Roadmap](roadmap.md)** — what is done and what is next.

## Guarantees

- **Pure Go, zero cgo.** Standard library only; cross-compiles everywhere.
- **Faithful to Facter's schema.** Aggregate structured facts plus flat legacy
  aliases.
- **100% test coverage** including error branches, enforced as a CI gate, green
  on Linux, macOS and Windows across the six 64-bit Go targets.
