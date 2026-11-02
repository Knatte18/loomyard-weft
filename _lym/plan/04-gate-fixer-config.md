# Card 4 — gate-fixer-config

**Create:**
- `plan:internal/gateslot#Config.FixerTimeout` -> `func (c Config) FixerTimeout() time.Duration`

**Edit:**
- `internal/gateslot#GateKey`
- `internal/gateslot#Config`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#decodeConfig`
- `internal/gateslot#decodeGates`
- `internal/gateslot/template.yaml`
- `internal/gateslot#TestLoadConfig`
- `internal/gateslot#TestTemplateConfigAndLimits`

**Extends:**
- `internal/gateslot/doc.go`

**Intent:**
Add the agent-call fixer's config to gate.yaml (D3).

- `GateKey` gains `Fixer string` (`yaml:"fixer"`), a model-spec.
  `decodeGates` refuses a non-empty `fixer` on any key but `card`, naming the key and the way forward: remove `fixer` from that key with `lym config gate` from the prime, since only the `card` key's agent call hands a red verdict to a fixer.
  It refuses a `fixer` that `modelspec.Parse` rejects, naming the key, the parse error and the way forward to set a model-spec such as `sonnet[medium]`, or `""` to turn the fixer off.
  Both refusals are `guidedError`s in the style of the existing scope and lint refusals.
  An empty `fixer` is the fixer off: the key behaves exactly as today.
- `Config` gains `FixerTimeoutMin int` (`yaml:"fixer_timeout_min"`); `decodeConfig` refuses a value below 1 with the way forward to set it with `lym config gate`.
  `ConfigSpec`'s `Categories` gains `fixer_timeout_min` as `CategoryCrossPair`, as `busy_quit_sec`, since every template unit of a hub-wide module needs a category and `configreg`'s template-coverage test refuses one without; `fixer` sits inside `gates`, which its `CategoryGuard` entry already covers.
  `FixerTimeout()` returns it as a duration; it bounds the wait of the four new callers' fixers, while webster's `fix-verify` keeps `recovery_timeout_min`.
- `template.yaml` sets `card: {tiers: [0], scope: card, lints: [comments, pattern], fixer: "sonnet[medium]"}`, adds `fixer_timeout_min: 30` with a comment naming what it bounds, and the gates comment names `fixer` as a key the `card` key alone may hold.
  Since `gates` merges key by key with a hub's file, a hub that tunes the `card` key's tiers keeps the template's `fixer`.
- `doc.go` gains a paragraph on `fixer` and `fixer_timeout_min`.

Tests, tier0:
- `TestLoadConfig` gains rows: the template's `card` key loads `fixer` `sonnet[medium]`; a hub `card` key that sets only its tiers keeps the template's `fixer`; a hub `card` key with `fixer: ""` loads it off; `fixer` on `webster` is refused naming `webster` and the way forward; an unparsable `fixer` on `card` is refused naming the key; `fixer_timeout_min: 0` is refused.
- `TestTemplateConfigAndLimits` asserts the template's `fixer_timeout_min` is 30 and `FixerTimeout()` is 30 minutes.

**ImpactSummary:** two new fields on gate.yaml's decoded config, both read only by later cards; no existing key's load changes.
