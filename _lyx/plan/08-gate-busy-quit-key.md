# Card 8 — Add busy_quit_sec to the hub-wide gate config

**Edit:**
- `internal/gateslot#Config`
- `internal/gateslot#Limits`
- `internal/gateslot#Config.Limits`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#decodeConfig`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestLoadConfig`
- `internal/gateslot#TestTemplateConfigAndLimits`
- `internal/hubgeom#TestWebsterGeometry`
- `contracts/specs/refusal-spec.md`

**Intent:**
D7's budget: `gate.yaml` gains `busy_quit_sec`, the continuous window of CPU-busy samples after which `lyx gate test`'s watchdog, which card 10 adds, sends a Go test binary one SIGQUIT.

- `Config` gains `BusyQuitSec int` with tag `yaml:"busy_quit_sec"`, and `Limits` gains `BusyQuit time.Duration`, which `Config.Limits` sets from it in seconds; 0 means the watchdog is off.
- `template.yaml` gains `busy_quit_sec: 120` with a trailing comment in the file's style: the seconds a Go test binary under `lyx gate test` may keep a CPU core at least 90 percent busy before it gets one SIGQUIT and dumps its goroutine stacks; 0 turns the watchdog off; raise it for a package that is CPU-bound but healthy.
- `ConfigSpec`'s categories gain `busy_quit_sec` as `configengine.CategoryCrossPair`, like the other keys, so it is hub-wide only and not a `LocalKeys` key (`PATTERN-hub-wide-config`).
- `decodeConfig` refuses a negative `busy_quit_sec` naming the key, in the wording of its other checks: `gate config key "busy_quit_sec": -1; want 0 or more`; 0 is accepted. `LoadConfig` appends its usual way forward, fixing the file with `lyx config gate` from the prime.

An absent key needs no code: `configengine.Load` fills a template key the hub file lacks, so a hub not yet reconciled reads 120 and still has the watchdog, and the binary-change reconcile writes the key into an existing hub's file.
`gate.yaml` is decoded with plain `yaml.Unmarshal`, so an older binary reading a file that carries the key ignores it.

Tests:
- `templateDefaults` gains `BusyQuitSec: 120`, so `TestLoadConfig`'s template row and `TestTemplateConfigAndLimits` pin the template value; `TestTemplateConfigAndLimits`'s wanted `Limits` gains `BusyQuit: 120 * time.Second`.
- `TestLoadConfig` gains three rows: `slots: 2\ngo_parallel: 4\ncli_wait_sec: 300\nbusy_quit_sec: 0\n` loads with `BusyQuitSec: 0`; the same file without the key loads with `BusyQuitSec: 120`; `busy_quit_sec: -1` is refused with `gate config key "busy_quit_sec": -1; want 0 or more; fix it with "lyx config gate" from the prime`.
- `TestWebsterGeometry` compares the whole `Limits` its fixture `gate.yaml` loads to, so its fixture gains `busy_quit_sec: 9` and its wanted `Limits` gains `BusyQuit: 9 * time.Second`.

Docs: `gateslot`'s package doc names `busy_quit_sec` among the module's keys, accepted at 0 and refused below it, and the refusal-spec rows "gate slot: gate.yaml absent or invalid" (both) and "gate.yaml unreadable or invalid" say "holds a value below 1, or a negative `busy_quit_sec`".

**ImpactSummary:** one new key on the hub-wide gate config and one field on `Limits`, read by `hubgeom`'s pool and `gatecli`; equality checks on `Limits` in `gateslot` and `hubgeom` tests are updated.

**Commit:** 8: busy_quit_sec in the hub-wide gate config
