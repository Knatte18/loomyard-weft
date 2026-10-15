---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: "_lyx/discussion/decision-record.md, D7 Bound (lines 93-95) and the second Open risks bullet (line 118)"
    summary: >-
      D7's Bound misstates what the session deny refuses (module-wide understates the subtree and all words,
      the tier clause reads as covering build and vet, the variable clause is missing),
      and the second Open risks bullet repeats the bound
    fix: >-
      In D7, replace the three Bound lines with a Bound that cites its source and states the real shapes.
      Write: Bound - the agents' session deny (`rawGoCommandPattern` in `internal/shuttleengine/claudeengine/settings.go`)
      refuses a `go test`, `go build` or `go vet` whose arguments hold a word containing `...` or the word `all`,
      a `go test` whose `-tags` value names `tmux` or `llm`,
      and any command naming the gate-slot, strand-name or prebuilt-binary environment variable.
      It is a guardrail on static text, so a run behind a script or `bash -c` passes too.
      Every other command a build skill names, such as `goimports -w`, `golangci-lint run`, `go install …@latest`,
      a `go test` of one package such as `./internal/x`, and every `dotnet build` or `dotnet test`,
      runs unrefused and outside the gate slots.
      Keep the closing sentence about the template comment, the operator's per-hub choice and the verify gate.
      In the Open risks section, shorten the second bullet so it points at D7's Bound instead of restating it.
      End it with - sends agents to run that skill's commands,
      which D7's Bound lists as unrefused where they are not module-wide or tier runs, outside the gate slots;
      D7's template comment is the only thing that steers the operator away from listing one.
  - id: F2
    severity: LOW
    class: consistency
    location: "_lyx/discussion/decision-record.md, D4 (lines 56-68)"
    summary: >-
      D4 does not say which wiring loads the language module for the merge conflict session,
      so a plan writer can place the load in landingshed or in the I/O-free landingDeps,
      and D4 says both once for that run invocation and once per construction
    fix: >-
      In D4's Decision, replace the sentence beginning "The merge conflict session's engine is built by `landingshed` at both Merge-In and Finalize"
      with: The merge conflict session's engine is built inside `landingshed` at both Merge-In and Finalize, so `landingshed` loads no config;
      `internal/loomcli`'s `wire()` loads the module beside `landing.yaml`,
      and the list reaches `landingshed.Deps` and then `mergeresolve.Deps` as a told value, with `landingDeps` staying free of I/O.
      Make the load timing one phrase in both places:
      in the Decision write "loads the `language` module once per process",
      and in the Rationale write "Loading once per process gives every spawn that process builds the same list."
      Add to Notes for the plan writer:
      `landingDeps` in `internal/loomcli/landingdeps.go` is called from `arm.go` and `sharedbootstrap.go`; both pass the list.
---

### [LOW] D7's Bound does not match the deny it describes

**Location:** `_lyx/discussion/decision-record.md` D7 Bound (lines 93-95) and the second `## Open risks` bullet (line 118).

**Issue:** D7 states a bound and what the hatch lets through, so the attack-surface question is answered.
The listed examples all hold against `rawGoCommandPattern` in `internal/shuttleengine/claudeengine/settings.go:131-133` and the pinned cases in `TestRawGoDenyHook_DeniesModuleWideAndTierRunsOnly`:
`goimports -w`, `golangci-lint run` and `go install …@latest` are not go test, build or vet, and a package-scoped `go test ./internal/x` passes (`package_scoped_test`).
`dotnet build` and `dotnet test` are not matched either.
The sentence that carries the bound is still inexact in three ways:

- "module-wide" understates the match.
  The deny matches any word containing `...` or the word `all`, so `go test ./internal/x/...` is refused (`subtree_pattern`) though it is not module-wide.
  The Decision's "a package-scoped `go test`" runs unrefused only for a literal package path.
- "`tmux`- or `llm`-tier `go test`, `go build` and `go vet`" reads as if the tier clause covers build and vet.
  Only `go test -tags` naming `tmux` or `llm` is refused; the `-tags` shape has no build or vet form.
- "refuses only" omits the third shape, any command naming the gate-slot, strand-name or prebuilt-binary variable.
  A build skill does not name those, so this does not change what the hatch lets through, but "only" is false as written.

The deny is also a guardrail on static text (`settings.go:128-130`): a run behind a script, `bash -c` or a `cd` plus a relative pattern passes.
D7 reads "refuses only ..." as the bound, and the Decision should say the refusal is not a barrier.

The second `## Open risks` bullet repeats the bound in "which the session settings mostly let through".
"Mostly" is right in aggregate, but it restates D7's Bound in a second place with a looser word.
Citing `rawGoCommandPattern` as the source also keeps the record from pinning a snapshot of another package's pattern.

### [LOW] D4 leaves the merge conflict session's load site ambiguous

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 56-68).

**Issue:** D4's first sentence puts the load in "the wiring that builds a code role's engine or its deps", and its second sentence says "`landingshed`'s deps carry the list from its wiring".
"Its wiring" has `landingshed` as its antecedent, and `landingshed` is a package, not a wiring layer.
The code has three candidate sites:

- `landingshed` itself calls `mergeresolve.New` from `NewMergeIn` and `NewFinalize` (`internal/landingshed/mergein.go:78`, `finalize.go:113`), and the package already owns `landingshed.LoadConfig`.
  A plan writer reading "built by `landingshed`" can load `language` there.
- `landingDeps` in `internal/loomcli/landingdeps.go:51` assembles `landingshed.Deps`, and its own comment says every argument arrives resolved and it does no I/O and returns no error, so a load placed there breaks that contract.
- `wire()` in `internal/loomcli/wiring.go:472` loads `landing.yaml` eagerly into `c.landingCfg` (`cli.go:84`), which both `landingDeps` call sites (`arm.go:429`, `sharedbootstrap.go:335`) then pass on.
  `wire()` runs for start, run and step but not for status or pause (`wireLightweight` loads no config), so a bad `language.yaml` refuses only the verbs that build producers.
  This is the site that fits D3's refusal and D4's "wiring".

The sentences do not contradict each other, but they leave the choice to the plan writer, and two of the three choices break an existing contract.
D4 also says the load happens "once for that run invocation" in the Decision and "once per construction" in the Rationale.
Merge-In and Finalize are two constructions, so the two phrases give different load counts when read literally.
`support-log.md` Q10 says "once per engine construction", so the log carries the looser phrase too; the record is the file the plan writer reads, so the fix is there.
