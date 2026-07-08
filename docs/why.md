# Why a Go engine, not a Ruby port

Puppet's Facter is Ruby, leans on C extensions, and shells out to many system
tools. That is fine on a full Ruby install, but the go-\* ecosystem runs Ruby
through **`rbgo`** — a pure-Go embedded Ruby (`go-embedded-ruby`) that ships as a
**static, cgo-free binary**. A fact engine that pulls in C extensions or a Ruby
runtime would defeat that.

## The split

go-facter draws a clean line:

- **The engine (this project, pure Go).** Discovering facts — reading
  `/etc/os-release`, parsing `/proc/meminfo`, enumerating interfaces, detecting a
  hypervisor — is deterministic system inspection. It lives here as pure Go and
  depends only on the standard library.
- **The Ruby surface (downstream, `go-ruby-facter`).** Ruby's `Facter.value` /
  `Facter[]` / `Facter.add` / `Facter.to_hash` and the custom-fact DSL are a thin
  mapping onto this engine's API. That binding lives in the consumer, next to
  `rbgo`.

> **This library resolves facts; the Ruby layer exposes them as Facter.**

## Fidelity over convenience

Fact **names and structure follow Facter's schema**, not a Go-idiomatic shape and
not `gopsutil`'s vocabulary. `os.family` is `"Debian"`, `os.release.full` is the
distribution version, `networking.interfaces.<name>.bindings` is the array Facter
publishes, and the flat legacy aliases (`operatingsystem`, `osfamily`,
`ipaddress`, …) exist alongside the aggregate facts. That fidelity is what lets a
Puppet manifest resolve the same facts it always has.

## Testability by construction

Every interaction with the operating system — file reads, command execution,
interface enumeration, the current user — flows through an **injectable seam**.
Collectors branch on a runtime `GOOS` string rather than build tags. Together
that means the per-OS collectors and *their error branches* are exercised
deterministically against fixture data, without root and independent of the host
the tests run on — which is how the project holds **100% coverage on Linux, macOS
and Windows** from a single test suite.
