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

### Hardware inventory — `dmi`

```text
dmi.manufacturer                                    system vendor (SMBIOS)
dmi.bios.vendor / version / release_date
dmi.board.manufacturer / product / serial_number / asset_tag
dmi.chassis.type / asset_tag
dmi.product.name / serial_number / uuid
```

Sourced from `/sys/class/dmi/id` on Linux, `system_profiler` / `ioreg` on
Darwin and `wmic` on Windows, and skipped where DMI is unavailable (many
containers). The `bios_*`, `board*`, `chassis*`, `manufacturer`, `productname`,
`serialnumber`, `uuid` legacy aliases derive from it.

### `ssh`, `selinux`, `load_averages`

```text
ssh.<algo>.type / key                               algo: rsa / dsa / ecdsa / ed25519
ssh.<algo>.fingerprints.sha1 / .sha256              SSHFP records, computed in pure Go
selinux.enabled / enforced / current_mode           Linux; reports {enabled: false} when absent
selinux.policy_version / config_mode / config_policy
load_averages.1m / 5m / 15m                          run-queue averages as floats
```

The `ssh<algo>key` and `sshfp_<algo>` flat facts derive from the structured
`ssh` fact, so each host key is read and fingerprinted at most once.

### Cloud, Ruby & agent facts

```text
cloud.provider                                       aws / gce / azure, from DMI (no network)
ec2_metadata.*                                       best-effort, non-blocking IMDS probe on EC2
ruby.version / sitedir / platform
fips_enabled                                         bool; /proc/sys/crypto/fips_enabled on Linux
aio_agent_version                                    puppet-agent AIO package version, when present
augeasversion                                        installed Augeas library version, when present
env_windows_installdir                               Windows install directory from the environment
```

The `cloud` / `ec2_metadata` probes are bounded and non-blocking: a host that is
not on a provider never stalls fact resolution.

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
| `bios_vendor` / `bios_version` / `bios_release_date` | `dmi.bios.*` |
| `boardmanufacturer` / `boardproductname` / `boardserialnumber` / `boardassettag` | `dmi.board.*` |
| `chassistype` / `chassisassettag` | `dmi.chassis.*` |
| `manufacturer` / `productname` / `serialnumber` / `uuid` | `dmi.manufacturer` / `dmi.product.*` |
| `ssh<algo>key` / `sshfp_<algo>` | `ssh.<algo>.*` |
| `memorysize_mb` / `memoryfree_mb` / `swapsize_mb` / `swapfree_mb` | `memory.*` (mebibytes) |

Aliases resolve lazily through the structured fact, so the underlying probe runs
at most once regardless of which shape is queried.
