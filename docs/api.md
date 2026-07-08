# Usage & API

```go
import "github.com/go-facter/facter"
```

## The Collection

`facter.New()` returns a `*Collection` with every built-in fact registered
against the real host. Facts resolve **lazily** and are **cached** for the life of
the collection.

```go
c := facter.New()
```

### Querying — the surface the Ruby layer maps onto

```go
func (c *Collection) Value(path string) (any, bool)         // Facter.value / Facter[]
func (c *Collection) ValueString(path string) (string, bool)
func (c *Collection) ToHash() map[string]any                // Facter.to_hash
func (c *Collection) Names() []string
```

`Value` takes a dotted path: the first segment names a top-level fact and further
segments descend into a structured fact's nested maps.

```go
name, ok := c.Value("os.name")                       // "Ubuntu", true
ip,   ok := c.Value("networking.interfaces.eth0.ip") // "10.0.0.5", true
_,    ok  = c.Value("os.nope")                        // nil, false
```

`ToHash` resolves every fact into one nested map (facts absent on this host are
omitted) — the whole inventory, ready to serialise.

### Extending — custom facts

```go
func (c *Collection) Add(name string, r Resolver)
func (c *Collection) AddFunc(name string, fn func(c *Collection) (any, bool))
func (c *Collection) AddValue(name string, value any)
```

These are the Go surface behind Ruby's `Facter.add`. A resolver may return
`(nil, false)` to declare the fact absent on this host, and may query other facts
through its `*Collection` argument.

```go
c.AddValue("role", "web")
c.AddFunc("uptime_minutes", func(cc *facter.Collection) (any, bool) {
    s, ok := cc.Value("system_uptime.seconds")
    if !ok { return nil, false }
    return s.(int64) / 60, true
})
```

### Encoding

```go
func facter.MarshalJSON(v any) (string, error) // Facter --json shape
func facter.MarshalYAML(v any) string          // Facter --yaml shape
```

Both are exported so the Ruby binding can reproduce Facter's output verbatim.

## Command line

```sh
facter                # print all facts as JSON
facter os.name        # print a single fact by dotted path
facter --yaml os      # YAML output of a structured fact
facter --json memory  # explicit JSON
```

A single-fact query that resolves prints the value; an absent fact prints nothing
and exits non-zero, matching `facter(8)`.
