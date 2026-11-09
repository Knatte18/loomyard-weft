```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Decisions walked**
- D1, held: `lym adopt payments` from a plain shell. The caller validates and spawns the worker, which takes `adopt.lock` and writes `running` at `reconcile`. The caller follows and exits 0 on `done`. In an agent session, `still running` at the cap exits non-zero, so `lym s` stops.
- D2, broken (F1): a worker killed after `owner`, re-run with the same subjects. The resume at `deploy` gets its inputs (moved runs, subjects whose owner write stood, paused slugs) from the record. But the end envelope is meant to be built from the outcomes of earlier processes' steps, and the record does not store them.
- D2, held: a worker killed after `pairs`. The re-run restarts at `reconcile`. The `pause` step skips the slugs the record lists, and the owner write follows a fresh reconcile.
- D3, held: `lym adopt root` with one project held back. The other subjects run through `check` and the record ends `stopped`. Resolving subjects before the reconcile matches today's `adopt` (`internal/adoptcli/adopt.go:227-231`).
- D4, held: `lym adopt root` on B, with beam X having no owner of its own. After the reconcile, X's owner read goes through `ParentOf` to root, which reads A. Root is in the subject set with no owner write, so X's watcher and dobby stay up and idle. Once the owner write lands, the board reads B and the suppression ends.
- D5, held: run R, which B's dobby had owner-paused (`OwnerCaller`). The `pause` step turns that entry into an `AdoptCaller` entry. After the owner write, `recheckOwnerPause` finds no `OwnerCaller` entry left and lifts nothing. `lym tender resume R` lifts the entry by scope (`internal/tenderengine/resume.go:111-129`), with no pair check.
- D6, held: the deploy fails after the owner write. The record stops at `deploy` and the runs stay adopt-paused. Only a front task can start, as D6 says.
- D7, held: adopt run from an orch strand. The caller sets `--no-orch` on the request, the record keeps it, and a resume from a plain shell keeps it too. The beam step starts the dobby only.
- D8, held: green end. The fix task is removed and nothing is lifted or started.
- D9, held: a `stopped` record. `lym beam status` shows the stop's way forward plus `lym adopt --discard`. `ReadCheckViews` skips non-directory entries (`internal/adoptcli/checkview.go:55`), so `adopt.json` beside the subject directories does not break it.
- D10, held: worker killed at `deploy`, then `lym s` in the prime. `--here` reads the record: it is `interrupted`, `check` is not done and root is listed. It takes the record's request and resumes at `deploy`. The caller waits to the end and exits 0, and only then `beam start` runs.
- D10, held: `lym s` in an owned-here worktree beside another subject's record. The record does not take the worktree up, so `--here` pulls Bolt, ends `owned_here`, and `beam start` runs.
- D11, held: idle old owner A. A probe pulls Bolt and reads B. A has no record, so the dobby owner-pauses its runs, pushes `archive-owner/` tags for the tips origin lacks, and exits. A failed archive keeps the dobby up and retries at the next probe.
- D12, held: `lym adopt --archives root` on B. It reads the `archive-owner/root/*` tags and the runs' tags per repository and compares each against that repository's branch of the slug. `archive/` tags never match the prefix.

**Findings**

F1 — MEDIUM, class `consistency`, location D2 (`_lym/discussion/decision-record.md:53-55`)
- **Summary:** the record must hold an end envelope "built from the outcomes of every step the record lists as done, earlier processes' steps included". But D2's list of what the record holds has no per-step outcomes: only steps done with their end times, the step in flight, the inputs later steps need, the state and the stop error.
- **Effect:** a worker that resumes at `deploy` after the first process died is not handed the reconcile's warnings, the per-slug outcomes from the pair add and the pause, or the owner write's result. The envelope D1's caller prints, and D10's `lym s` relays, cannot cover those steps.
- **Evidence:** today's envelope carries the reconcile, deploy, per-slug outcomes, beam steps and checks (`internal/adoptcli/adopt.go:117-135`, and the help text at `internal/adoptcli/cli.go`: "The envelope carries per slug what happened, the deploy, the beam steps, the checks…").
- **Fix:** D2's field list should name each done step's outcome as stored in the record.

**Tool calls**
- `internal/tenderengine/resume.go:95-135` and `Held` greps in `internal/dobbyengine`: to check that a hand resume lifts a slug's entry with no pair check, and how the dobby reads `Held`. The manifest had only resume's function index.
- `internal/adoptcli/adopt.go:208-240` and `620-660`: to check whether today's subject resolution runs before or after the reconcile, against D2's "record written before any step". The manifest had only the function index.
