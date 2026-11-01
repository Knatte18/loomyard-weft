# Card 3 — keep-warm-resolver

**Create:**
- `internal/shuttleengines#KeepWarmAnswer`
- `internal/shuttleengines#KeepWarm`
- `internal/shuttleengines#keepWarmWith`
- `internal/shuttleengines#baseRole`
- `internal/shuttleengines#TestKeepWarm`

**Edit:**
- `internal/shuttleengines#allowedImports`

**Extends:**
- `//internal/shuttleengines/doc.go`

**Uses:**
- `internal/shuttleengines#build`
- `internal/shuttleengines#registry`
- `internal/shuttleengine#LoadConfig`
- `internal/agentname#Parse`
- `internal/agentname#MatchesRole`
- `internal/testkit/shuttlefake#Engine`

**Intent:** D4's one keep-warm resolver, the only route by which a cap reaches `internal/gatecli` and `internal/shuttlecli` (PATTERN-shuttle-provider-seam).

`KeepWarmAnswer` holds `Cap` and `LongCap` (`time.Duration`), `MaxReruns` (`int`, shuttle.yaml's `keep_warm_max_reruns`) and `Fallback` (`error`, nil when the answer came from the engine).
`KeepWarm(dirs, strand)` is `keepWarmWith` over the registry's constructors.
`keepWarmWith` parses `strand` with `agentname.Parse`, takes its role through `baseRole`, loads shuttle.yaml with `shuttleengine.LoadConfig(dirs, "shuttle")`, builds the set with `build(cfg, dirs, constructors)`, and asks the engine `cfg.Engine` names for `KeepWarmCaps(base)`; the answer carries `cfg.KeepWarmMaxReruns`.
A strand that does not parse (an empty one included), a shuttle.yaml that does not load, a set that does not build, and a `KeepWarmCaps` error each answer the fallback: `Cap` and `LongCap` both `shuttleengine.DefaultAwaitCap`, `MaxReruns` 0, so no long wait is taken, and `Fallback` the cause.
The resolver never logs: the caller logs a fallback at its own level (D4: the gate at Warn, `lym shuttle await` at Debug).
`baseRole(role)` drops a numbered suffix: for `role` of the form `<base>-<N>` where `agentname.MatchesRole(role, base)` holds, it answers `base`; any other role answers itself, so `implementer-2` resolves as `implementer` and `role-x` stays `role-x`.

`allowedImports` in the package's seam test gains `internal/agentname`, the one new import (PATTERN-agent-name: the strand is parsed there alone).
The doc names `KeepWarm` as the one keep-warm resolver, the numbered-role rule and the fallback.

`TestKeepWarm`, tier0, rows over a temp anchor:
- the template answers 250s, 55m and MaxReruns 10 for strand `sh:slug:implementer`, with no fallback;
- a local claude layer with `keep_warm_cap_roles: {implementer: 120s}` answers 120s for `sh:slug:implementer-2` and for `sh:implementer` (a prime strand), and 250s for `sh:slug:implementer-x`'s unmatched role;
- strand `""` and strand `not a strand` answer the fallback, `DefaultAwaitCap` with a non-nil `Fallback`;
- an invalid claude local layer (a `binary` key) answers the fallback;
- a shuttle local layer setting `engine: bogus` answers the fallback;
- `keepWarmWith` over a constructor map whose `claude` entry builds a `shuttlefake.Engine` answers a zero `Cap` with no fallback, the answer under which a gate call runs in-process (D2).
`baseRole` is reached through those rows and gets no test of its own.

**ImpactSummary:** a new read-only resolver and one import added to the seam test's allowlist; no existing caller changes.
