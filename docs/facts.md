# Facts & schema

go-facter resolves the Facter **aggregate (structured)** facts and the **flat
legacy aliases** that derive from them. Structured facts are nested maps queried
by dotted path (`os.release.full`); legacy aliases are top-level scalars
(`operatingsystemrelease`). A fact that a platform cannot provide in pure Go is
skipped gracefully rather than reported wrong.

## Aggregate facts

### `os`

```text
os.name              e.g. "Ubuntu", "Darwin", "windows"
os.family            e.g. "Debian", "RedHat", "Suse", "Darwin", "windows"
os.release.full      distribution / kernel version, and .major / .minor
os.architecture      e.g. "x86_64", "aarch64"
os.hardware          machine hardware name
os.distro.*          Linux: id, codename, description, release.*   (from /etc/os-release)
os.macosx.*          Darwin: product, build, version.full/major/minor  (from sw_vers)
```

### `kernel`

`kernel` (`Linux` / `Darwin` / `windows`), `kernelrelease` (full release string),
`kernelversion` (numeric prefix, e.g. `6.5.0`), `kernelmajversion` (`6.5`).

### `networking`

```text
networking.hostname / fqdn / domain
networking.primary                     elected primary interface name
networking.ip / ip6 / mac / netmask    summary of the primary interface
networking.interfaces.<name>.ip        per-interface ip / ip6 / mac / mtu
                             .netmask / .network
                             .bindings[]   {address, netmask, network}
                             .bindings6[]
```

### `processors`, `memory`

```text
processors.count / physicalcount / models[] / isa
memory.system.total / total_bytes / available / available_bytes / used / capacity
memory.swap.*                           (same shape; omitted when there is no swap)
```

Byte counts are integers; the human strings (`total`, `used`, …) use Facter's
binary units (`15.51 GiB`), and `capacity` is a `"NN.NN%"` string.

### Storage

`mountpoints.<path>` (device, filesystem, options, size/used/available bytes +
human + capacity), `filesystems` (comma-separated on-disk fs types), and
`disks.<name>` (size, model, vendor). These are Linux facts, parsed from
`/proc/mounts`, `df`, `/proc/filesystems` and `/sys/block`.

### Virtualization, uptime, identity

```text
virtual / is_virtual     hypervisor (DMI + cpu flag) and container (docker/podman/cgroup) detection
system_uptime.seconds / hours / days / uptime
identity.user / group / uid / gid / privileged
timezone, path, facterversion
```

## Legacy aliases

| Alias | Resolves to |
|-------|-------------|
| `operatingsystem` | `os.name` |
| `osfamily` | `os.family` |
| `operatingsystemrelease` / `operatingsystemmajrelease` | `os.release.full` / `.major` |
| `architecture` / `hardwaremodel` | `os.architecture` / `os.hardware` |
| `hostname` / `domain` / `fqdn` | `networking.*` |
| `ipaddress` / `ipaddress6` / `macaddress` / `netmask` | primary interface |
| `interfaces` | comma-separated interface names |
| `processorcount` / `physicalprocessorcount` | `processors.count` / `.physicalcount` |
| `memorysize` / `memoryfree` / `swapsize` / `swapfree` | `memory.*` |
| `uptime` / `uptime_seconds` / `uptime_days` / `uptime_hours` | `system_uptime.*` |
| `id` / `gid` | `identity.user` / `.gid` |

Aliases resolve lazily through the structured fact, so the underlying probe runs
at most once regardless of which shape is queried.
