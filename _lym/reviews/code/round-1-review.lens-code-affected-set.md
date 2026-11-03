```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Result: no findings.**

I checked each thing the diff forces outside its own lines. The diff is one 21-line section added to `contracts/orchrole/orch-role.md`.

| Change | Set found | Covered? |
|---|---|---|
| Role body text (the embedded bytes) | `contracts/stencils/markers_test.go:36` pins only the `command_index` and `decision_note_title` markers, and the section adds neither. `registry_test.go:20` and `:144` keep the role a fixed stencil, and `roleskills_test.go` and `parentdirective_test.go` scan the role body for load verbs and parent and edit markers. The new section has none of these. | Yes, nothing needs updating. |
| Names `lym fabric add`, hand deletion and `lym batten cancel ... --remove` | The no-denied-recovery scan is `cmd/lym/norecoverydeny_test.go`. It covers only `contracts/specs`, `contracts/stencils`, `plugins/ly/skills` and the webster, shed and loom Go literals. `contracts/orchrole` is outside it. `lym fabric add` is in no denied-form list (`internal/shellguard/rows.go` Forms, and `operatorDeniedForms`, which has only `git reset --hard`, `git push --force`, `git push -f` and `rm -rf`). The section's "never" lines are also behind the pinned `never ` marker. | Yes. The role's header claim "names no denied recovery command" still holds. |
| Commands the section tells the orch to use | These are `lym beam create`, `lym beam start`, `lym board upsert`, `lym batten status` and `lym batten cancel --remove`. All exist; I read `beam create` and `beam start` at `internal/beamcli/cli.go:474` and `target.go:87`, and `batten cancel --remove` at `internal/battencli/cancelcmd.go:130`. The role's command list is the `{{.command_index}}` marker, generated from the operator-audience annotation, so no hand-kept list goes stale. Role lines 282-283 already document `beam start` and `beam status`, and role step 3 already says `--remove` is needed before a kept pair's slug runs again. | Yes. |
| Vocabulary | The section says "worktree" throughout and never "pair", as the working rule requires. | Yes. |
| `PATTERN.md` and `pattern/` | `PATTERN-stencil-ownership` is the only entry that touches the role. The edit keeps the role in `contracts/orchrole` with no board copy, and it restates no other producer's format. No declared PATTERN changes were given in the directive. | Yes. |
| Commit message | There is one commit, with no `Pattern-Departure:` trailer, and nothing in the diff needs one. | Yes. |
| Docs mirroring the role | A search for the role's section names and `orch-role` in non-test Go, `.md` and `docs/` found no doc that mirrors the role's sections. The hits are `internal/orchengine/prompt.go` and `doc.go`, which render the file whole, plus unrelated research reports. | Yes. |

Two notes outside my lens, for the handler:
- The new section's `lym batten status <slug>`, "filtered to its current producer" claim looks wrong. `internal/battencli/arm.go:944-955` shows the status envelope carries batten's own `current_producer`, which is a row of the batten recipe (`contracts/recipes/batten-recipe.yaml:41-84`, ending in `Run-Shed`). The inner producer (tapestry row or `Beam-Up`) is not in that envelope.
- Run-loop item 4 (`orch-role.md` around line 58) says "never read `lym batten status` on your own". The new section tells the orch to run it, which is in tension unless the section is read as a stop-triggered read.

Tool calls (4):
1. Searched `internal/orchengine`, `shellguard` and `agentsettings` for denied-command text and docs mirroring the role. The manifest lacked the deny tables and any section-name mirrors.
2. Searched the shellguard `Forms` and the no-denied-recovery tests. I needed to know whether `lym fabric add` or hand deletion is a denied form.
3. Read `shellguard/rows.go:320-345` and `cmd/lym/norecoverydeny_test.go:1-60`. I needed the scan's surface and its denied-form list, to see whether `contracts/orchrole` is scanned.
4. Read `contracts/stencils/parentdirective_test.go:90-130`. I needed to see how the role body is scanned.
