```yaml
lens: code-affected-set
outcome: reported
```

Task: judge what the diff forces outside its own lines for the code-affected-set lens. The list below gives each change and the set I found.

**Per change, set found**
- `Board.SetStatusReleasingProject` (new). Its only production caller is the `set-status` verb at `internal/boardcli/cli.go`. The other `SetStatus` callers are unchanged and release no member: `internal/pairteardown/claim.go:181`, `internal/loomcli/landingdeps.go:210`, `internal/fronttask/hub.go:47`, `internal/battencli/cancelcmd.go:319`, `internal/beamcli/deps.go:237` and `internal/boardengine/board.go:227`. Covered.
- `set-status` success envelope gains a `notice`. `outputSuccessWithNotice` exists at `cli.go:1235` and `strings` is imported. The `internal/boardcli/doc.go` change and the verb's help text both describe the notice. Covered.
- `set-status` help text and the `doc.go` files in `internal/boardcli` and `internal/boardengine`. These describe the code and agree with it. Covered.
- Plugin and contract docs that tell a reader how to release a project or task: `plugins/ly/skills/board/SKILL.md:44`, `plugins/ly/skills/board-policy/SKILL.md:38`, `contracts/orchrole/orch-role.md:38` and `docs/getting-started.md:37`. Each describes a task release, which is unchanged. None describes a project release. Whether they need a mention is a judgment call, so I raise nothing.
- PATTERN entries: the diff touches none, the record declares no PATTERN changes, and the commit messages carry no `Pattern-Departure:` trailer. Nothing to land.
- Stencils: the diff changes none.

**Findings**

1. **LOW, consistency.** `internal/boardcli/verbs_test.go` (new `stepProjectRelease` at about lines 553–566, inserted at line 552). The new step and its doc comment sit between `stepProjects`'s doc comment (lines 546–551) and the `func stepProjects` line. The `stepProjects` doc comment is now attached to `stepProjectRelease`'s comment block, and `stepProjects` has no doc comment of its own. The `stepProjects` comment opens "stepProjects drives the project kind…" and runs on directly into "stepProjectRelease pins…", so both steps are documented wrongly. Fix: move `stepProjectRelease` and its comment below `stepProjects`, or above its comment block, so each doc comment sits directly above its own function. Evidence: the diff hunk of `verbs_test.go` and my read of lines 540–556.

Nothing else turned up in this lens.

**Tool calls (3)**
1. `grep` for `outputSuccessWithNotice`, the `strings` import, doc files that mention `set-status`, and non-test `.SetStatus(` callers. My context lacked the caller set and the helper's definition.
2. `grep` over both `doc.go` files and a `sed` of `verbs_test.go` lines 540–556. This confirmed the doc-consistency claims and the misplaced comment.
3. `grep` of the plugin skills, contracts and docs for `set-status` wording, plus a `ls` of `internal/boardcli` and a `grep` of PATTERN files for release rules. My context lacked these doc surfaces.
