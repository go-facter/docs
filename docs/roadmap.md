# Roadmap

## Done — the engine

- **Resolver framework** — lazy, per-run-cached facts; dotted-path `Value`,
  `ToHash`, `Names`; `Add` / `AddFunc` / `AddValue` custom facts.
- **Core fact groups** — `os` (incl. `os.macosx`), `kernel*`, `networking`
  (interfaces + bindings + primary), `processors`, `memory` (+swap),
  `mountpoints`, `filesystems`, `disks`, `virtual` / `is_virtual`,
  `system_uptime`, `timezone`, `identity`, `path`, `facterversion`, plus the flat
  legacy aliases.
- **Facter 4 parity facts** — `dmi` (bios / board / chassis / product, with the
  `bios_*` / `productname` / `serialnumber` / `uuid` aliases), `ssh` host keys +
  SSHFP fingerprints, `selinux`, `load_averages`, `cloud` / `ec2_metadata`,
  `ruby`, `fips_enabled`, `aio_agent_version`, `augeasversion`,
  `env_windows_installdir`.
- **External facts** — `facts.d`: JSON / YAML / txt + executable facts.
- **CLI** — `facter` full dump or single dotted-path query, JSON / YAML.
- **Quality** — 100% coverage including error branches, `gofmt` + `go vet` clean,
  CI green on Linux, macOS and Windows across the six 64-bit Go targets (amd64,
  arm64, riscv64, loong64, ppc64le, s390x).

## Next

- **`go-ruby-facter`** — the Ruby binding: maps `Facter.value` / `Facter[]` /
  `Facter.add` / `Facter.to_hash` and the custom-fact DSL onto this engine, so
  Puppet manifests and Ruby code under `rbgo` resolve facts. Owned by a separate
  effort; this engine's stable API is its contract.
- **Deeper per-OS facts** — Windows memory / uptime without cgo, richer
  cloud-provider metadata beyond the base `cloud` / `ec2_metadata` facts, and
  structured `partitions`.
- **External-fact caching (TTL)** to mirror Facter's cached custom facts.
