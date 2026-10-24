```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked which pairs of the record's decisions act on the same state, then ran their failure and restart paths against each other.

**Pairs checked**
- D2 closure / D3 fallback: project ownership and the no-.csproj fallback agree. Held.
- D3 dotnet fallback / D5 inert_paths: inert paths are dropped and an all-inert diff gives the empty scope. Held.
- D4 per-project narrowed runs / D6 whole-solution build with `--no-build`: held.
- D5 inert carry / D6 build declaration in the fingerprint: the carry needs the current fingerprint. Held.
- D5 inert carry / D8 `carry: false`: excluded, as stated. Held.
- D5 carried or empty tiers / D6 "no build when every opted-in tier is carried or empty": held.
- D5 empty scope / D8 timing tier: **broken** (F1).
- D5 none-scoper diff / Constraint "go and none unchanged without the new keys": **broken** (F1).
- D1/D4 dotnet narrowed passes / existing green record writer and cover logic: **unspecified, Go-shaped** (F2).
- D6 build `{args}` / D8 timing args after `--`: **broken** (F3).
- D6 build timeout / verifytree's `FindTierTimeout` and the landing failure record: **unspecified** (F4).
- D6 build failure / the gate's exit-code rule: **unspecified** (F5).
- D6 build timeout / the tier's own `timeout_min` clock: **ambiguous** (F6).
- D6 build run record / `WriteRunRecord` naming by tier number and the `FailedTier` reader: **unspecified** (F7).
- D5 inert empty scope under go / today's "only the guard step runs" on an empty scope: **unspecified** (F8).
- D6 build inside a `slots: all` timing hold (D8): held.
- D7 warm nodes / D6 build env carrying the MSBuild server variable: held.
- D9 "verified tree carries both tiers" / D6 "carried tiers need no build": held.
- AC "inert-only diff runs no tier under each scoper" / a go hub with empty inert_paths: held.
- AC "whole pass of two vstest tiers carries both" / D1 dotnet green write: held only if whole entries are written as today (see F2).

**Findings**

**F1 — MEDIUM — Decisions 5 and 8, Constraints.** An empty changed set counts as "all inert".
- D5 says a computed or `--base` scope whose changed paths are all inert is the empty scope, under every scoper, and the `none` scoper now reads the diff to check this.
- An empty diff makes that condition vacuously true. Example: HEAD equals the merge base on a clean tree, with no green ancestor.
- So under `none` a computed form that runs whole today would run nothing. This happens even with empty `inert_paths`, which breaks the Constraint that `none` is unchanged without the new keys.
- D8's timing tier is hit too: it writes no green entry, so its base is the parent merge base. A call without `--all` on a docs-only or empty diff runs nothing.
- Under `dotnet` the timing tier is narrowed by the closure, or emptied when no class is selected.
- Fixes: the inert rule should need at least one changed path and should apply only when `inert_paths` is non-empty. D8 should say the timing call uses `--all` (or explicit targets), or that a `carry: false` / manual tier always runs whole.

**F2 — MEDIUM — Decisions 1 and 4 against the green record.** What a narrowed dotnet pass writes and carries is never stated.
- The existing writer records the packages a narrowed pass ran.
- It decides "whole" by expanding `all_targets` with `go list` under the tier's tags. That cannot expand a `.sln`.
- Its remainder logic re-runs a remainder as explicit packages.
- The record says nothing about dotnet. Either narrowed dotnet passes record nothing and only whole entries carry, as under `none`, or the record defines project-level entries.
- The AC "vstest tiers carry both" depends on the whole-entry path, and that path is Go-shaped.

**F3 — MEDIUM — Decisions 6 and 8.** The build's optional `{args}` collides with the tiers' args.
- The words after `--` fill each tier's `{args}`.
- D6 lets the build command carry `{args}` but never says which words fill it.
- If they are the call's words after `--`, a timing call's BenchmarkDotNet args, or `-- --filter X`, reach `dotnet build` and break it.
- The existing refusal for words after `--` on a command lacking `{args}` is also undefined for the build.
- Fix: drop `{args}` from the build, or state it never receives the call's words.

**F4 — MEDIUM — Decision 6 against verifytree and the landing failure record.** How a build timeout is reported is not stated.
- `verifytree.Verify` sets `TimedOut` only when the log holds `gateslot.TierTimeoutLine`. That line names a `tiers.tierN` rewrite as the way forward.
- A build timeout needs either its own line, which Verify also recognises, or the tier line pointing at `build=`.
- Otherwise a timed-out build reads as a plain failure, with a wrong or missing way forward.

**F5 — LOW — Decision 6 against the exit codes.** The exit code is undefined.
- Opted-in tiers that fail "without running" after a failed build have no stated exit code.
- The rule "first failing run's own code in plan order" doesn't say whether the build's code or the first skipped tier's code wins.

**F6 — LOW — Decision 6 against tier timeouts.** The two timeout clocks may overlap.
- The tier's `timeout_min` is counted from its slot hold and today also bounds `prebuilt_lyx`.
- D6 counts the build's own `timeout_min` from the same hold.
- It is unstated whether the tier's clock starts after the build, or whether the build also eats into the tier's budget.

**F7 — LOW — Decision 6 against the run log.** The build's record fits no existing name.
- `WriteRunRecord` names files `<stamp>-tier<N>-<pid>`, and the build has no tier number.
- `FailedTier` and the summary-line readers expect a tier.
- The record and log naming for scope form `build` need stating, so `lyx gate report` and the landing failure trace don't misread them.

**F8 — LOW — Decision 5 under the go scoper.** The guard step on an inert-only diff is unspecified.
- Today a Markdown-only empty scope under `go` still runs the guard step.
- D5 doesn't say whether an inert-only empty scope plans the guard step. For a Go hub that sets `inert_paths`, guard behaviour could change silently.

No further tool calls; my context held everything this lens needed.
