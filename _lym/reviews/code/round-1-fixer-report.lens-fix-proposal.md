```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the reviewer's proposals the fixer carried out (F1–F7).

1. **LOW**: `internal/dobbyengine/template.yaml` `autonomy` comment and `internal/beamsetting/beamsetting.go` (F5).
   The proposal named only the template, so three siblings still give the old two-item list and now contradict the changed text: "only for a severe halt or a merge-ready beam".
   Evidence:
   - `internal/dobbyengine/tasksetting.go:50`
   - `internal/dobbyengine/config.go:33`
   - `internal/heddleengine/glossary.md:27` (the `autonomy` glossary entry)

2. **LOW**: `contracts/stencils/raddle/raddle-template-agent.md`, the added "When no operator is present … into the proposal instead" lines (F3).
   The proposed `contract.takes` field is not a private note channel. `internal/loomshed/raddlefile.go:435` and `:453` copy it into the filed entry's body as `Takes: …`.
   A note such as "file X by hand" or "the entries land under project P" would then sit in a filed task's scope text, where that task's agents read it as an input.
   `cost_line` is specified as "one line saying what the figure rests on" (`contracts/specs/raddle-proposal-spec.md`), as is `next_best.line`, so neither is meant for such notes either.
   In auto mode nothing reviews the proposal, so a "file by hand" note still reaches no one there. The added text only helps the perch.

3. **LOW**: `contracts/orchrole/orch-role.md`, the added paragraph under Notices (F4).
   It sits after the list introduced by "Only these reach you:" and then says the raddle line "reaches you too", so the section contradicts itself.
   The proposal said to add the line to that list; as a fourth bullet it would agree.

4. **LOW**: `internal/runprofile/template.yaml` header (F2).
   The comment now says each profile holds "one block per recipe, tapestry, motif, darn, pawl, sampler or raddle", but the same file still has `arras:` blocks and `defaults.arras`.
   A user reading `profiles.yaml` sees a block the header says does not exist, and nothing explains it. The proposal asked for both the removal and keeping the keys.
   This is weak: hiding the name from comments is what was asked, so disputing or leaving it is defensible.

I found no breaking case for F1, F6 or F7. The only production caller of `UpsertTasksBatch` is `internal/boardcli/cli.go:717`, so reading the prior entry from the store is right.

Tool calls (2 of 6):
- Grep for `UpsertTasksBatch(` callers and for "merge-ready" siblings: my context had no caller list, and the earlier grep was cut off by `head`.
- Read the proposal spec's field list, grepped uses of `Takes`, `CostLine` and `NextBest.Line`, and grepped the raddle-review rubric: my context had none of these.
