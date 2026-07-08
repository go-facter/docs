# Custom & external facts

go-facter extends its fact set two ways: **custom facts** registered in-process
through the Go API, and **external facts** loaded from `facts.d` directories.
Together they are the seam the Ruby custom-fact DSL binds onto.

## Custom facts (in-process)

```go
c.AddValue("datacenter", "eu-west-1")
c.AddFunc("kernel_is_recent", func(cc *facter.Collection) (any, bool) {
    v, ok := cc.Value("kernelmajversion")
    return ok && v.(string) >= "6.0", true
})
```

Registering a fact replaces any earlier registration of the same name and
invalidates its cached value. A resolver returning `(nil, false)` marks the fact
absent. This is the mapping target for Ruby's:

```ruby
Facter.add(:datacenter) { setcode { "eu-west-1" } }
```

## External facts (`facts.d`)

```go
err := c.LoadExternalFacts("/etc/facter/facts.d", "/opt/puppetlabs/facts.d")
```

Each directory is scanned and every fact file is registered as a top-level fact.
Four formats are understood:

| File | Parsed as |
|------|-----------|
| `*.json` | a JSON object of facts |
| `*.yaml` / `*.yml` | a YAML mapping (nested maps, scalar sequences, scalars) |
| `*.txt` | `key=value` lines, with scalar type inference |
| any other file with an **executable** bit | run; its `key=value` stdout parsed as facts |

Directories, and non-executable files with an unknown extension, are ignored.
Errors reading a directory or an individual fact file are collected — the first
is returned, but every readable fact is still loaded — so one malformed file
never hides the rest.

```text
# /etc/facter/facts.d/site.txt
role=web
tier=frontend
```

```yaml
# /etc/facter/facts.d/app.yaml
app:
  name: checkout
  version: 3
tags:
  - critical
  - pci
```
