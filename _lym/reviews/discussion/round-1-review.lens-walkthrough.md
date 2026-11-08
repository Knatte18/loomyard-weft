```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end against the code and checked the end state against the decision's own stated purpose.

**Decisions walked**
- **D1**, an arras run resumed after Discussion approval: the purpose is met. Raddle-Write's `RecordPath` is `loomengine.DiscussionDecisionRecord`, the same path Discussion-Write writes (internal/loomcli/raddlecontext.go:119).
- **D2**, a Raddle-File send-back, then a fresh Raddle-Write, then the perch: the purpose is met as stated, but the per-mode return handling it relies on is unstated (finding 2).
- **D3**, a proposal that leaves out an Added after Discussion entry, reviewed by the rubric: the purpose is met.
- **D4**, an arras run with `raddle_auto_approve` off: Raddle-File's `verify` matches the approval hash only in interactive mode (internal/loomshed/raddlefile.go:200), so a non-interactive mode files with no hash match. The filing step is met; Raddle-Write's end step is unstated (finding 2).
- **D5**, a one-part `tasks` proposal: `fileRaddleEntry` drops the part's dependency on `RunEntry` (internal/boardengine/raddlefile.go:155), the part is filed draft, and step 6 runs `EndRaddleEntry`. The purpose is met.
- **D6**, a three-part family whose project slug is the arras slug: `filing` builds `ProjectEntry` with `proposal.Project.Title`, `labels(proposal.Project.Type)` and a body made only of the prefix and the slugs (internal/loomshed/raddlefile.go:440-447). `fileRaddleEntry` upserts that onto the existing slug, so the title, type label and former body are replaced. That differs from D6's "keeps its slug, title and module labels … followed by the entry's former body". D6 states the end state, so this is plan work, not a record defect. The retry path holds: `isRaddleFiledFrom` matches the converted project's prefix (internal/boardengine/raddlefile.go:224).
- **D7**, a decision added after the filing on a family's part: the way forward misses its purpose (finding 3). The case of a decision added at Raddle-Write after a return is met.
- **D8**, a null status set on the converted project: its members are drafts, the release empties their status, and dobby's `released` and `heldByEntry` gate them by `depends_on` (internal/dobbyengine/starts.go:61-69). The purpose is met.
- **D9**: prose only, so there is no mechanism to walk.
- **D10**, a dependent X on the arras slug in the family outcome: the purpose is already met without D10 (finding 1). In the one-task outcome the purpose is met.

**Findings**

1. **MEDIUM, design, D10 (family outcome): D10's premise does not hold for a family.**
   - With D10 not applied, X keeps `depends_on: [<arras slug>]`, and that slug becomes a plain project whose members are draft tasks.
   - `Dependencies` counts a plain project as done only when its status is done, or when every landing member is done (internal/boardengine/dependency.go:10). siblings.go:40 confirms that a dependency on a project waits on each landing member.
   - So X does not start before the family lands, and the rationale "a dependent left on it would start before any of the work it waits on has landed" is false here.
   - In this outcome D10's re-pointing changes nothing, yet it adds the filing-failure path ("a run lock on a dependent") to every family filing.
   - Evidence: internal/boardengine/dependency.go:10, internal/boardengine/siblings.go:40.

2. **LOW, design, D4/D2: `perch` mode is not mapped onto Raddle-Write's existing per-mode branches.**
   - Today those branches split `auto` from everything else. `clearLastAttempt` moves the proposal to `ReturnedPath` only in `RaddleModeAuto` and removes the approval file otherwise (internal/loomshed/raddlewrite.go:136-140). The cold proposal is stored by the gate's onPass in auto mode and by the propose verb in interactive mode (internal/loomshed/raddlewrite.go:37).
   - D4 says `lym loom raddle-approve` refuses in `perch` mode, so the interactive way of ending the session and storing the cold proposal is gone. D4 never says that `perch` mode ends the session and stores the cold proposal the way auto mode does, or which return handling D2's re-armed cycle uses.
   - Walked with today's code, `perch` falls into the non-auto branch on a return. It keeps the returned proposal in place, and Raddle-File's stage needs a cold proposal that nothing in `perch` mode stores.
   - Evidence: internal/loomshed/raddlewrite.go:37, internal/loomshed/raddlewrite.go:134-140, and `stageColdFile` read in internal/loomshed/raddlefile.go:379-384.

3. **LOW, design, D7 (after-filing way forward): the way forward points at a run that does not exist.**
   - After the filing, the refusal "names the filed task's own run". In the family outcome there are several filed tasks, and the record does not say which one the refusal names.
   - Each filed part is a draft with no run until the operator releases it. So `lym loom decision add` on a part fails with the existing "run `lym loom start`" refusal (internal/loomcli/decision.go:71), which does not lead anywhere useful for an unreleased draft.
   - The way forward does not reach a place where the decision can be recorded before release.
   - Evidence: internal/loomcli/decision.go:71, and internal/boardengine/raddlefile.go:82 ("The parts are filed draft").

**Tool calls**
- Listed Raddle-File's and Raddle-Write's mode handling, because my context lacked how the shed rows branch on mode.
- Read Raddle-File's `Call`, `verify`, `stageAndFile` and `filing`, because my context lacked how the filing is built from the proposal and the entry.
- Grepped the `RecordPath` wiring and read `clearLastAttempt`, to check D1's same-record-path claim and the per-mode return handling.
