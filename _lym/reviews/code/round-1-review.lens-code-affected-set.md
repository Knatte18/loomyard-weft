```yaml
lens: code-affected-set
outcome: reported
```

**Per change — affected set, covered or not**

1. **Decision verbs admit every orch on the run's beam chain** (`decisionCallerRefusal`, `decisionStoreVerb`, `decisionOrchChain`, `onChain`):
   - `internal/loomcli/doc.go` — updated, covered.
   - `docs/overview.md:1255` — still states the old rule, **not covered** (F1).
   - `internal/hubgeom/doc.go:46-47` package doc — still describes the old guard, **not covered** (F2).
   - decision tests' fake — gains `orchChain`, covered.
2. **`parent-review answer` admits the chain**: `internal/loomcli/doc.go` — covered; `docs/overview.md:1498` names the verb without a caller rule — still accurate; `parentReviewFixture` deps — covered.
3. **Tender caller rules for a beam's orch** (`callerOf`, `refuseCaller`, `pairUnderBeam`, `refuseReapCaller` text):
   - `internal/tendercli/doc.go` — covered.
   - `docs/overview.md:896` — **not covered** (F3).
   - `contracts/orchrole/role-beams.md:137,153` (beam orch running `lym tender resume <slug>`) — now consistent, covered.
   - the changed reap text — asserted nowhere but the new test's "ask your parent session" substring.
4. **`parent-directive-handback` stencil + `parentdirective.HandBack`**:
   - `contracts/stencils/stencils.go` registry and embed — covered.
   - tackler and arbiter stencil header comments — covered.
   - `internal/parentdirective/doc.go` — covered.
   - `RenderTacklerPrompt` and `shedarbiter.Prompt` doc comments — covered.
   - tackler stencil body (lines 83, 90, 91) and arbiter stencil body (lines 62–131) say "the run's parent" — agree with the new directive.
   - the hand-back text's own claim about where a stop goes — **not covered** (F4).
5. **`hubgeom.OrchChain` and `hubgeom.ParentOrchName`**: `hubgeom/doc.go` entries added — covered; one added sentence documents a consumer (F5).
6. **Batten and loom wiring of `ParentName`** (`tacklerDepsFor` signature): its one test caller `driverdeps_test.go:113` — updated, covered. The driver prompt (`internal/loomcli/driverprompt.go:59`) is unchanged and already takes `c.parentName` — the change description's claim holds.
7. **Declared PATTERN changes**: none declared in the directive — nothing to land.
8. **Commit messages**: the three code commits each carry a `Root-Cause:` trailer naming the defect they remove; no `Pattern-Departure:` or `Loosened-Check:` trailer; the two rewrap commits change comments only.

**Findings**

- **F1 — LOW · consistency · `docs/overview.md:1255`** — The overview still says `decision add` "refuses a named session other than the run's parent orch, the nearest live orch up its beam chain"; the code now admits every orch on the chain, read with no liveness probe. Evidence: `internal/loomcli/decision.go:783-792`; `docs/overview.md:1255`.
- **F2 — LOW · consistency · `internal/hubgeom/doc.go:46-47`** — The package doc still reads "The `lym loom decision add --by parent` guard admits the destination of notices instead, so the orch that receives them can record the call." The guard now admits the `OrchChain` list, not the notice destination. Evidence: `internal/hubgeom/doc.go:46-47`; `internal/loomcli/decision.go:1893-1899`.
- **F3 — LOW · consistency · `docs/overview.md:896`** — The tender rules still read "An agent may pause, resume or revive only its own pair; the orchestrator and an external caller may do each for the hub or any pair." This omits the new beam-orch rule (its beam and the pairs under it; refused reap and hub-wide calls), and the reap line at `:894` ("the orch and an external caller may run it") leaves whether a beam's orch counts as "the orch" open. Evidence: `docs/overview.md:894,896`; `internal/tendercli/cli.go:60-123`.
- **F4 — LOW · consistency · `contracts/stencils/parent/parent-directive-handback.md:9`** — The hand-back text tells the agent "a stop the run hands back reaches `{{.parent_name}}`", but a stop's halt notice goes to the nearest live orch: `orchcli.NotifyParent` for loom (`internal/orchcli/notify.go:50-53`, wired at `internal/loomcli/wiring.go:254,978`) and `NotifyParentHalt`/`NotifyRunFromHalt` for batten (`internal/battencli/notices.go:30-51`). When the beam's orch reads as down, or its pair is absent, the stop reaches the prime's orch, so the prompt's claim is false in exactly the case issue #231 describes. `ParentOrchName` can also name a removed pair's orch (`internal/hubgeom/parentof_integration_test.go` row `RecordNamingARemovedParentPair` → `tst:po-removed:orch`), which no notice reaches.
- **F5 — NIT · consistency · `internal/hubgeom/doc.go` (added `OrchChain` paragraph)** — "A caller check that admits the orchs above a run reads it, so an orch that reads as down is still admitted for its own runs" documents a consumer outside the package; `PATTERN-code-conventions`: a package's `doc.go` documents only that package. Evidence: the diff's `internal/hubgeom/doc.go` hunk; the PATTERN-code-conventions entry in the constraints.

Tool calls:
1. grep for the old and new reap refusal texts, caller rules in `contracts/specs`, tender and decision mentions in `contracts/orchrole/role-beams.md`, and tests over the parent-directive registry — the context lacked whether any other site asserts or states these.
2. grep of `docs/overview.md` and `contracts/specs` for the tackler or arbiter parent wording and the `parent-review answer` admission, and of the Go sources for the agent-own-pair text — the context lacked those doc lines beyond the two overview spans already read.
