---
verdict: BLOCKING
rationale: "All eight round-1 findings are fixed in the artifacts, but the record never decides how standalone-mode strands are named: lyx webster/burler run --target-dir boot their own reed session through standalonegeom.ReedGeometry with no hub, weft or .lyx-code, while the record makes the code mandatory for every reed strand and names hubgeom.ReedGeometry as the one teller"
---

## Round-1 findings: all resolved

I checked each fix against the current artifacts, not only against the fixer report.

- **F1:** the Constraints bullet now states what happens to `strand_name`: the key leaves `Config` and both templates, a stale key loads and is ignored, and `lyx config reconcile` removes it. A `reed config` acceptance criterion pins this.
  This matches the code: `internal/reedengine/config.go` loads `reed.yaml` with a plain `yaml.Unmarshal`.
- **F2:** the Mirrors decision, the `custom-title` open risk and the provider criterion now treat these as separate cases: no `custom-title` entry counts as drift, and a missing or unreadable transcript is logged and counts as no drift.
  The open risk now names the real degradation: the watchdog retypes `/rename` and logs it on every idle tick.
- **F3:** the hand-started hub bullet now says the env-prefix path is not a deliverable of this task.
- **F4:** Parentage names the default run as run-id `self`, which `shedrun` resolves to the slug, so its seed is `_lyx/shed/<slug>/seed.json`.
  The Auto-mode assumption and the reed criterion now agree with it.
- **F5:** support-log turn 15 now points to the hub review, Q13 gives the revised resolution, and the reason for rejecting `--parent` is now the env prefix.
- **F6:** a criterion now covers the spawn refusal for a worktree name that fails the slug grammar, and the docs criterion now names the reed and fabric package docs.
- **F7:** the redundant criterion for the hand-started hub seed is gone.
- **F8:** both dated attributions are dropped, and both rationales keep their substance.

The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject.
I agree with the report's two "not a defect" calls: reusing a freed `-N` is settled by the "lowest free" rule, and the manifest-retire line in Scope is a constraint the plan writer needs.

## New finding: standalone-mode naming is undecided

**Where:** `decision-record.md`, the Goal, Short code ("A spawn on a hub with no recorded code refuses"), Name grammar, Mirrors ("reed is told the name prefix … through its geometry, built by `hubgeom.ReedGeometry`"), and Notes ("`hubgeom.ReedGeometry` is the one teller to extend").

**Issue:** lyx has a second reed geometry teller.
`CONSTRAINTS.md` names `internal/hubgeom` and `internal/standalonegeom` as the only `Geometry` constructors.
`docs/overview.md` documents standalone mode: `lyx webster run --target-dir <repo>`, a plain git checkout with no hub, "boots its own private reed session in-process".
Burler wires through the same path (`internal/burlercli/wiring.go`, `internal/cliwire/standalone.go`).
A standalone target has no weft:main and no `.lyx-code`, it has no task worktree slug in fabric's sense, and it has no shed default-run seed to take a parent from.

The Goal says every strand lyx spawns through reed carries `<code>:…`, and the grammar makes the code mandatory.
The record decides refusal only "on a hub", and it never mentions standalone mode.
The plan writer is left with three readings, and each one breaks something or invents a decision:

- Standalone spawns refuse for lack of a code. That kills standalone webster and burler.
- Standalone strands keep today's template names. That contradicts retiring `strand_name`.
- Standalone strands get some fallback code. That contradicts the rejected repo-name fallback.

The Out list excludes processes reed does not spawn, but standalone strands are spawned by reed.

**Fix:** add a decision on how standalone mode forms names, or that it is out of scope and what reed does for it then.
Possible answers include a fixed sentinel code, a code derived from the standalone hash8, or excluding standalone sessions from the grammar with their existing naming kept as a named exception.
The decision should say which teller supplies the prefix, whether `LYX_PARENT` is ever set there, and how the `strand_name` retirement applies.
It also needs an acceptance criterion: a standalone `webster run` still spawns its strands.
