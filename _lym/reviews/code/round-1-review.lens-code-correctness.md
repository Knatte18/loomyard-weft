```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function, branch and error path the diff adds or changes.

**Cases run**
- `restoreStatusPaneBeforeWake`, seam nil (hand-built receiver): returns `""` and the resume goes on. Correct.
- `restoreStatusPaneBeforeWake`, seam returns nil: returns `""`, so no `status_pane`. Correct.
- `restoreStatusPaneBeforeWake`, seam returns `errors.New("reed up failed")`: Warn is logged and the warning names the error and `lym loom start`. Correct.
- `wire`'s `restoreStatusPane` closure, `reedEngine.Up()` fails (boot-config refusal, preflight failure): the error is returned and the envelope carries `status_pane`. Correct.
- Closure, Up succeeds and `ensureStatusStrand`'s `c.reed.Status()` fails: the error is returned and `status_pane` is set. Correct.
- Closure, status strand absent and `AddStrand` fails: the error is returned and `status_pane` is set. Correct.
- Closure after a reboot: Up boots, `clearAllPaneBindings` leaves the persisted status strand not-live, and `resolveStatusStrandAction` returns `statusStrandReplace`. If `ReplaceStrand` then fails, `ensureStatusStrand` logs and returns nil, so the resume reports a clean success with no `status_pane`. **Not correct** (finding 1).
- Closure, status strand live and current: Keep, nil. Correct.
- Closure, `c.recipe` for the window title on resume: `resolveRecipe` sets it during arming for every verb (`arm.go:107-114`). Correct.
- Closure, warm session on the parked path: Up does not boot and clears no bindings. A replace of a stale status strand leaves the tracked driver strand alone. `resumeParkedDriver` then sees the same driver GUID. Correct.
- Closure, booted session on the Go-spawn path: the cleared bindings affect only strands that are already dead. `spawnGoDriver` reads none of the reed state that was cleared. Correct.
- `resumeFromRunState`, Go-spawn branch, restore fails and `spawnGoDriver` also fails: the refusal is reported and `paneWarning` is dropped from the envelope, though it is logged. A re-run of resume runs the restore again. Partly correct (finding 2).
- `resumeFromRunState`, parked branch, restore fails and the delivery is busy or fails: same as the case above, `paneWarning` is dropped from the refusal. Partly correct (finding 2).
- `resumeFromRunState`, done, awaiting not resumable, running with a live driver or lock, run lock held, driver lock held, not parked, unfinished merge: none of these reach a restore. That matches the summary's claim. Correct.
- `succeedAfterWake`, `paneWarning` non-empty: it adds `status_pane` to the fields map, which each call site builds fresh. Correct.
- The warning's way forward, `lym loom start` over a resumed run:
  - with a just-spawned Go driver holding its driver lock, `mustSpawn` is false, no marker is present and the state is not halted, so start succeeds and leaves the driver alone (`start.go:391-417`);
  - with a woken LLM driver whose strand is live and whose marker is gone, start reaches the same branch.
  The claim "leaves the live driver working" holds. Correct.

**Findings**

1. **MEDIUM, design**, `internal/loomcli/wiring.go:1384-1389` (the `restoreStatusPane` closure) together with `internal/loomcli/sharedbootstrap.go:363-369`.
   - **Summary:** On the reboot path the restore goes through the replace branch, and a `ReplaceStrand` failure there is swallowed. Resume then reports success with no `status_pane`, and the operator gets neither the pane back nor the notice the entry asks for.
   - **How it happens:** `lifecycle.go:725-727` clears every pane binding on a booted Up, so the persisted status strand reads not-live. `bootstrap.go:231-232` maps a found but not-current strand to Replace. `sharedbootstrap.go:366-369` logs the `ReplaceStrand` error and returns nil.
   - **Contradicted claims:** `doc.go:112`, the `resumeCmd` Long text ("a restore that fails … the envelope's "status_pane" names the failure"), and the summary's "The failure is logged at Warn, and the success envelope gains a `status_pane` field".

2. **LOW, design**, `internal/loomcli/resume.go:147-153` and `resume.go:174-182`.
   - **Summary:** When the restore fails and the wake that follows is then refused, `paneWarning` is dropped. The refusal envelope never mentions the missing pane; only the log has it.
   - **How it happens:** `paneWarning` is read only inside `succeedAfterWake` (`resume.go:60-65`). The `spawnGoDriver` error, `ErrSessionBusy` and other delivery errors go through `refuse`, which ignores it.
   - **Why only LOW:** every refusal on these paths names `lym loom resume` again or another verb that reruns the restore.

Tool calls:
1. grep for `c.recipe` assignments and `resumeCmd` wiring: the manifest did not show whether the recipe used in the status window title is set on the resume path.
2. Read `arm.go:95-120` (`resolveRecipe`) and the `arm`/`armAt` call sites: to confirm that arming sets the recipe for every verb, resume included.
