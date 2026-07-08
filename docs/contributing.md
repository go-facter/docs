# Contributing

## Principles

- **Pure Go, zero cgo.** Only the standard library. No shelling out that a file
  read can replace; every OS probe goes through an injectable seam.
- **Faithful to Facter.** Match Facter's fact names and structure, not
  `gopsutil`'s and not a Go-idiomatic shape. When in doubt, check what Facter 4
  emits.
- **100% coverage, including error branches.** New collectors add their parsing
  as pure functions and wire OS access through seams, so tests cover both the
  happy path and every failure path with fixture injection — no root, any OS.

## Working locally

```sh
git clone https://github.com/go-facter/facter
cd facter

go vet ./...
go build ./...

COVERPKG=$(go list ./... | paste -sd, -)
go test -race -coverpkg="$COVERPKG" -coverprofile=cover.out ./...
go tool cover -func=cover.out | tail -1   # must read 100.0%
```

The coverage gate runs on Linux, macOS and Windows, and the suite also runs under
qemu on the four non-native 64-bit arches. Because collectors branch on a runtime
`GOOS` string (not build tags) and read through seams, a new fact group's logic
is exercised on every runner from one set of fixture-driven tests.

## License

BSD-3-Clause. By contributing you agree your work is licensed under it.
Copyright the go-facter/facter authors.
