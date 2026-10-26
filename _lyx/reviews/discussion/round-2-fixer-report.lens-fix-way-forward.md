```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that every refusal, halt, notice and criterion the fix introduced can be carried out from the state it names and makes progress.

**[MEDIUM] The Finalize halt's ways forward fail for a member whose run is in flight on another machine.**
- **Changed text:** D9's Finalize bullet: "halts naming, for each member, only `lyx batten cancel <member> --remove` or moving its `project` off the beam with `lyx board upsert`, then `lyx loom resume` in the beam worktree, never landing the member". The matching acceptance criterion says the same.
- **Summary:** For a member with no pair on this machine and a run in flight elsewhere, the named upsert is refused here. The named cancel works only on the machine that holds the member's pair, and the halt does not say so. The fix also removed the old "land it where it runs" way forward, so that member is left with no way forward the halt names.
- **Evidence:**
  - `internal/boardengine/runlock.go:76-81`: `checkRunLocks` refuses any `scopeChanged` edit of a run-held entry.
  - `scopeChanged` compares `before.Project != now.Project` (`runlock.go:105`, `:125`), so a run-held member's `project` cannot be moved off the beam.
  - The removed wording is today's no-pair blocker in `internal/battencli/beamland.go`: "move its `project` off the beam with `lyx board upsert`, or land it where it runs".
- **Second, lesser gap in the same text:** the halt is posted to the beam's own orch and names no directory for the cancel. `lyx batten cancel` resolves through `ResolveBattenPrime` (`internal/battencli/cancelcmd.go:103`). That function refuses inside a beam whose entry holds a run status (`internal/battencli/refusal.go:80-86`), which is the case from Parent-Align on. Its refusal does name the parent worktree, so this case is reachable with one extra hop, and the halt could name the parent or prime as where to run the cancel.

No other findings:
- **Run-lock refusal of a re-parent, naming `lyx batten cancel <slug>`:** at Beam-Await `ResolveBattenPrime` passes inside the beam because no hold stands. A cancel refused for members names those members, so the chain makes progress.
- **Cancel's reconcile refusal:** it names the reconcile's own way forward.
- **D6 bound's two adopts:** each one leaves a single machine owning both the beam and its parent, which is the resume rule in D6.
- **Driver-dead notice naming `lyx loom start` in the beam worktree:** I found no loomcli refusal of a beam worktree. The grep came back empty, so this is unconfirmed rather than proven.
- **Marker removal on interrupted-landing re-entry:** it is reachable from the state it names.
- **F10's repeated `--anchor`:** it can be carried out.
- **New acceptance criteria:** each names a state that the decisions make reachable.

Tool calls beyond context:
1. Grepped `loomStartRemedy`, loom resume and loomcli beam refusals, to check that the new notice's `lyx loom start` and the halt's `lyx loom resume` exist and are not refused in a beam.
2. Grepped `ResolveBattenPrime` callers and the cancel entry points, to see whether cancel runs under the beam's hold.
3. Read `ResolveBattenPrime`'s body in `refusal.go`, for the landing refusal's wording and its redirect.
4. Grepped loom start for beam handling and read `RunLockedError`'s message, to check the run-lock way forward and whether a member's `project` move is refused.
5. Read `checkProjectField` in `project.go`, to see whether moving a member off a held project is refused.
