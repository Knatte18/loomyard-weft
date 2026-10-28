```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing path that writes or reads the state the record's decisions introduce or change.

**Paths traced, per decision**

- **D1, D2 against `lym loom decision add`.** `internal/discussionparser/append.go:119` writes `### Added after Discussion (…)` headings, and D1 exempts those. D2 does not check them, and the record keeps `decision add` on `Validate` alone. Survives.
- **D1 against carry-over blocks.** `internal/discussionparser/carryover.go:23` puts these blocks in `## Open risks`, and D1 reads only `## Decisions`. Survives.
- **D2 against the darn gate's commit and the darn verb's commit.** At `internal/loomshed/gates.go:165` and `internal/loomcli/validate.go:66-72`, a D1 or D2 finding makes the check unclean, so nothing commits. The history check keeps its `Validate`-clean precondition as D2 states. Survives.
- **D2 against webster's rendering of added entries.** `internal/websterengine/doc.go:246` only reads the entries, and D1 and D2 change nothing about them. Survives.
- **D4: Env fields read by the discussion gate.** `internal/loomcli/wiring.go:296` fills `WorktreeRoot` and `wiring.go:309` fills `PlanIndex` for every recipe, before `entries_gate.go:155` builds `NewDiscussionGate`. The record's claim that Env "already holds both" is right. Survives.
- **D4: the `validate-discussion` verb.** It reads the same `c.env` (`internal/loomcli/validate.go:66-72`), so the verb and the gate stay on one function. Survives.
- **D4: quarry outage handling.** Outside planglyph, only webster and the CLI verbs branch on `ErrQuarryUnavailable` (`websterengine/runlevel.go:668`, `loomcli/validate.go:298`, `webstercli/validate.go:244`). The loomshed gates return the error as an infrastructure fault, and D4's Bound says the same for the discussion gate. Survives.
  The `validate-discussion` verb reports it through its generic `loom: validate discussion: <err>` line, and the wrapped message names quarry, so it is not misread as an invalid record.
- **D5 against `impactset`'s guard scan.** `internal/impactset/guards.go:185` refuses a marker that is not directly above the func line, unless only keep lines sit between. D5 states that the guard scan will admit seam lines. Survives, by D5's own stated change.
- **D5 against the `testtiming` keep scanner.** `cmd/testtiming/keep.go:14,41-43` already skips any `//lym:` lines between a keep line and its func line, so a stacked `//lym:seam` passes there unchanged. Survives.
- **D9: the gate-side `PlanContext` builder.** `planContextOf` (`internal/shedrecipe/entries_gate.go:125-132`) reads Env fields filled at wiring, so tier tags "as `RefuseRaw` is" means a new Env field filled the same way. Survives.
- **D9: the verb-side `PlanContext` builder.** `internal/loomcli/validate.go:256-277` reads `gate.yaml` through `gateslot.LoadConfig`, treats an absent config as `ErrConfigAbsent`, and is skipped under `--require-approved`. This matches D7's "no `gate.yaml` skips" and D9's `--require-approved` exclusion. Survives.
- **D9: done-card skipping.** `ValidatePlan` hands only non-done cards to `planContextFindings` (`internal/loomshed/gates.go:262-269`). `ValidateReworkPlan` hands all cards of the fresh generation (`gates.go:330`), which holds no done card. Both match D9's claim. Survives.
- **D9: webster dispatch.** `beginbatch` and `runlevel` reach `index.ValidateDispatch` and never `planContextFindings`, which matches "webster's dispatch do not run them". Survives.
- **D8 through D4's method in `planContextFindings`.** Both callers (`gates.go:269`, `gates.go:330`) already hold `index` as a parameter, as D9 claims. Survives.

**Findings**

None. Every existing path I traced keeps the decisions' results, or the record already states the change that path needs.

**Tool calls**

1. grep for carry-over and added-decision writers: the manifest lacked who writes the regions D1 reads.
2. `internal/loomcli/validate.go` discussion verb and `wiring.go:295-315`: the manifest lacked the verb's Env fill for the root and index.
3. Both `PlanContext` builders (`entries_gate.go:105-140`, `validate.go:240-300`): needed for D9's "both builders fill the same way" claim.
4. grep for `ErrQuarryUnavailable` outside planglyph and planindex: needed to see who reads the outage that D4's new path returns.
5. grep for `testtiming:keep` readers: needed to find a second scanner of the comment block that D5 stacks into.
6. `cmd/testtiming/keep.go`: needed to check whether that scanner admits a `//lym:seam` line.
