```yaml
lens: code-correctness
outcome: reported
```

**Per function or branch — case run — verdict**

- `hubgeom.chainWorktrees`: member `m→b→prime` → `[m b ""]` ✔; gone parent (`found=false`) → `[m ""]` ✔; cycle `a→b→a` → `[a b ""]` ✔; start `""` → `[""]` ✔; read error at `b` → nil, err ✔.
- `hubgeom.OrchChain`: no shortname → `nil, nil` + warning; every caller then falls through to the parent-based refusal, as before ✔. From the prime: `walkStart` → `""` → `[prime orch]` ✔. Origin read failure further up the chain → whole call errors (see F2).
- `hubgeom.walkStart`: `isPrimeWorktree` error returned, as the old inline code did ✔.
- `hubgeom.ParentOrchName`: pair forked from the prime → `ParentOf` "" → `tst:orch` ✔; slug with no record and no board entry → `tst:orch` ✔; removed parent pair → names `tst:po-removed:orch`, an orch that no longer exists (see F1); `ParentOf` read error returned (see F3 for the caller).
- `decisionOrchChain`: task member → `[member beam prime]` minus own → `[beam prime]` ✔; beam project record → `[beam …prime]` kept whole ✔; prime run → the prime's chain entry has `Worktree ""`, which ≠ the prime's own worktree name, so the prime orch stays, matching the old `NearestLiveParent` answer ✔; nil (no shortname) → `DeleteFunc` on nil safe ✔.
- `decisionCallerRefusal`: chain error + no sampler error → "parent could not be resolved" text ✔; chain error + sampler error → sampler text ✔ (as before); caller on chain → admitted ✔; off chain + sampler error → sampler text ✔; off chain + `parent()` fails → unresolved text ✔; off chain + empty parent name → refused with operator way forward ✔. Correct except the chain-error regression (F2).
- `decisionStoreVerb`: same cases ✔; the arbiter strand still skips ✔. Except F2.
- `parentAnswerCallerRefusal`: caller = `asked.Strand` with chain error → admitted first ✔; empty chain (no shortname) → "neither" refusal, matching old empty `parent.Name` ✔; chain error → "could not be resolved" ✔ (except F2).
- `parentReviewAnswerVerb`: `parent()` now runs after the caller check; its error feeds only `parentAnswerRefusalText` ✔.
- `tendercli.callerOf`: `""` → external; `hub:orch` → orch; `hub:b:orch` → beam `b`; `hub:t:darn` → agent slug `t`; unparsable → agent slug `""` ✔. `oth:b:orch` from another hub → beam `b` (shortname unchecked; the orch case was already unchecked before the diff — no finding).
- `refuseCaller`: beam orch slug `""` → refused ✔; beam orch naming a member or the beam itself → chain holds the beam → admitted ✔; naming an outside pair → refused ✔; `underBeam` error → "re-run" refusal ✔; agent own pair → admitted, other → refused ✔; orch and external → pass ✔.
- `pairUnderBeam`: absent pair → false → refused ✔; naming the prime's name → `OrchChain(prime)` = `[""]` → false ✔; pair recorded under a beam gone here → walk ends → false ✔; `Stat` error other than not-exist → falls through to `ResolveWorktree`, which surfaces it ✔.
- `refuseReapCaller`: beam orch refused; orch and external pass ✔. Text "only the hub's orch" omits the external caller, who also may run reap — wording only, no finding.
- Tender `pause`/`resume`/`revive` verbs: the caller check now runs after cwd resolution, so a cwd error shows first — harmless ✔.
- `parentdirective.HandBack`: `""` → none stencil ✔; name → fills `parent_name` ✔; missing hand-back stencil → error naming it ✔ (the stencil is seeded in the per-process pre-run, so an existing hub gets it).
- `RenderTacklerPrompt`, `shedarbiter.Prompt`, loom wiring of `c.parentName`: `c.parentName` is set in `wire()` before the driver and arbiter deps are built ✔; rendered text see F1.
- `battencli` `loadedTacklerDeps` / `loadedArbiterDeps`: a `ParentOrchName` error is returned (see F3).

**Findings**

- **F1 — MEDIUM, design — `contracts/stencils/parent/parent-directive-handback.md:9` (with `internal/hubgeom/parentof.go:22`, `internal/loomcli/wiring.go:685`)** — The hand-back directive says "a stop the run hands back reaches `{{.parent_name}}`", where the name is the recorded parent (`ResolveParent`/`ParentOf`), but the hand-back's halt notice goes through `NotifyParentHalt`/`NotifyRunFromHalt` to the nearest *live* orch (`internal/orchcli/notify.go` `NotifyParent`, `NearestLiveParent`). When the beam's orch reads as down, or the recorded parent pair was removed (`RecordNamingARemovedParentPair` → `tst:po-removed:orch`), the prompt names one orch while the stop reaches another. Evidence: `parentof_integration_test.go:425`; `orchroute/walk.go` `FromPair` → `From` → `NearestLiveOrch`.
- **F2 — LOW, design — `internal/loomcli/decision.go:783-789`, `internal/loomcli/decision.go:1313-1316`, `internal/loomcli/parentreview.go:206-207`** — The new caller checks call `OrchChain`, which reads every origin record up to the prime, where the old `NearestLiveOrch` stopped at the first live orch. So an unreadable origin record above the nearest live orch now refuses, with "the run's parent could not be resolved", the very orch the old code admitted. The chain error is checked before the old `parent()` check runs, so admission is not a superset of the old rule on this path, and the summary's "every caller admitted before is still admitted" fails here. Evidence: `nearestlive.go` (`chainWorktrees` returns on a `parentOf` error; `nearestLive` returns at the first live worktree).
- **F3 — LOW, design — `internal/battencli/drive.go:497-500`, `internal/battencli/drive.go:571-574`** — A `ParentOrchName` error (a `ParentOf` read failure: origin record, worktree list, or the board when the slug has no pair) now fails the whole tackler or arbiter deps. The driver's `tackle` then hands the stop back with "the tackler could not judge the stop" (`internal/sheddriver/drive.go:557-558`), so a batten stop that used to be tackled halts. The parent name is an optional prompt line; hubgeom's own convention for prompt parent names is `parentNameOrEmpty` (warn, render no parent; `internal/hubgeom/parent.go`), but here it has become a hard dependency of the recovery path.

Tool calls:
1. grep `internal/sheddriver` for the `Tackler(` call site — context lacked how the driver handles a tackler error.
2. read `internal/sheddriver/drive.go:540-569` (`tackle` failure branch) — same reason.
