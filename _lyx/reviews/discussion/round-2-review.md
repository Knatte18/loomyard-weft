---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 Rationale (Bound) and Open risks second bullet
    summary: D7's bound claims the session settings refuse a listed build skill's commands, but the deny covers only module-wide or tmux/llm-tier go test/build/vet, so goimports -w, golangci-lint, go install @latest, package-scoped go test and every dotnet build/test run unrefused outside the gate slots
    fix: In decision-record.md D7, replace the Bound sentence with one that states the real bound - the agents' session deny refuses only module-wide and tmux- or llm-tier go test, build and vet; every other command a build skill names (goimports -w, golangci-lint run, go install ...@latest, a package-scoped go test, and every dotnet build or dotnet test, which no deny matches) runs unrefused and outside the gate slots; what bounds it is the template comment and the operator's per-hub choice, and the verify gate still runs the hub's own build, so no gate is skipped. Rewrite the second Open risks bullet to match - an operator who lists a build skill in a hub that builds through lyx gate sends agents to run that skill's commands, which the session settings mostly let through, outside the gate slots; D7's template comment is the only thing that steers the operator away from listing one.
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D4 Decision, first two sentences
    summary: D4 says whatever constructs a code role's engine loads the module, then names landingshed as the conflict session's constructor while saying landingshed is told the list by its wiring and does not load it
    fix: In decision-record.md D4, reword the first sentence to say the wiring that builds a code role's engine or its deps loads the language module once for that run invocation and tells the engine the list; keep the second sentence's statement that landingshed's deps carry the list from its wiring, and drop the phrase calling landingshed the loading constructor.
---

### [MEDIUM] D7's bound overstates what the session settings refuse

**Location:** `_lyx/discussion/decision-record.md` D7 Rationale (`Bound:`) and the second `## Open risks` bullet.

**Issue:** D7 lets an operator list any skill, a build skill included, and states its bound as "in a hub that builds through `lyx gate`, the agents' session settings refuse those commands …, so the cost is wasted turns".
The Open risks bullet repeats it: "sends agents toward commands the session settings refuse; each refused command costs a turn".
The only matching deny is `rawGoCommandPattern` in `internal/shuttleengine/claudeengine/settings.go`: a `go test`, `go build` or `go vet` whose arguments hold `...` or `all`, a `go test` whose `-tags` names `tmux` or `llm`, and the lyx-owned environment variables.
Its own doc says "any raw package-scoped go test" passes.
`scribe:golang-build` also tells an agent to run `goimports -w <changed-files>`, `golangci-lint run`, and `go install golang.org/x/tools/cmd/goimports@latest` and the golangci-lint install, none of which any deny matches.
`scribe:csharp-build`, which D7 itself names, runs `dotnet build` and `dotnet test`, and no deny names `dotnet` at all, so in a .NET hub building through `lyx gate` every command that skill names runs, outside the `internal/gateslot` slots.
The "never a skipped gate" half holds, since the verify gate runs the hub's build regardless.
The "wasted turns" half is false: the stated bound is not what bounds the hatch, which the rubric's attack-surface test asks D7 to state truthfully.
The bound came from round 1's F3 fix, whose proposed wording carried the same overstatement; this finding corrects that wording and does not reopen F3's ask for a stated bound.
Severity is MEDIUM rather than BLOCKING because the template comment already steers operators away from listing a build skill, and the template default lists none.

### [NIT] D4 names landingshed as the loading constructor, then says it is told the list

**Location:** `_lyx/discussion/decision-record.md` D4 Decision.

**Issue:** The first sentence says "Whatever constructs a code role's engine loads the `language` module once at construction".
The second says the merge conflict session's constructor is `landingshed`, "so `landingshed`'s deps carry the list from its wiring", which means `landingshed` does not load it; `mergeresolve.New` is called from `internal/landingshed/mergein.go` and `internal/landingshed/finalize.go` with deps the loom wiring builds.
A plan writer reading the first sentence literally could put the config load inside `landingshed`.

## Focus verdicts

- D4 load timing across processes: not a defect beyond F2.
  The master's `RunDeps` are built in `internal/loomcli/wiring.go` and `internal/webstercli/run.go`, the recovery strand's `RecoverDeps` in `internal/webstercli/recoverbatch.go`; D4's last sentence makes each process load the list itself, so nothing is threaded across processes, and "an edit reaches the next spawn built by a process started after it" matches a `lyx loom` process that builds its deps once in `wire`.
- D7 and Open risks: defect, F1. The two agree with each other, but both state a bound the deny rules do not provide.
- Open risks and D3 on a misspelled skill: consistent.
  `internal/shuttleengine/claudeengine/skillload.go` classifies a skill the latest skill listing does not name as unknown, and `internal/shuttleengine/run.go` logs `shuttle: skill skipped` with cause `unknown` and no retry, which matches both "skipped at once with a logged warning" and D3's "never fails or hangs the launch".
- Acceptance criteria against D5 and Scope: consistent.
  The recipes hold Burler rows for Discussion, Plan, Webster and Merge in `contracts/recipes/loom-recipe.yaml` and Darn in `contracts/recipes/darn-recipe.yaml`; the criterion's positive set (Webster, Merge, Darn and their fix rows) and excluded set (Discussion, Plan and their fix rows) partition them exactly as D5 and Scope do.
