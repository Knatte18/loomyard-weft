```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, here is the set of other places that read or assert it, and whether the diff covers them.

**Per-change list**
- **`PullRequestMergeIn` in `normalizeTask`**
  - Set found: `boardcli` help (updated), `boardengine/doc.go` (updated), the `Task` field comment (updated), `task_test.go` (updated), and the `normalizeTask` doc comment (not updated).
  - Other readers: `boardengine/replay.go:199`, `runlock.go:210` and `sketch.go:59` treat the field as an opaque string, and `loomcli/landingdeps.go:361` passes it to the resolver. All are fine.
  - Not covered: the `normalizeTask` doc comment.
- **`ResolvePullRequest` and `TrueMerge`**
  - Set found: `PRReview.decide` (an empty `Open` takes `closeRecordedPullRequest`, so it is fine), and `beamcli/land.go:238` and `beamcli/pr.go:357` (they pass `""` as the field, so they are unaffected).
  - Not covered: `recordedBeamLanding` and `landingForDecision` (see F2). `landingshed/doc.go` and `PATTERN-landing.md` are updated.
- **Suffix move to `darnengine`**
  - The only references to the old names are in `beamcli/mergeparent.go` and the new `darnengine/mergetask.go`, and `mergeparent.go` now uses the new ones. Covered.
- **Roleguard exception**
  - `fabriccli/doc.go` and the file header are updated.
- **Imports**
  - `fabriccli` and `beamcli` now import `darnengine`. I found no enforcement test that forbids this. `PATTERN-fabric-boundary` restricts fabric engines, not the CLI package, so I raise nothing.
  - The `landingshed` allow-list entry is added.

**Findings**

- **F1, LOW, consistency.**
  - Location: `internal/boardengine/task.go:~202`.
  - Summary: the `normalizeTask` doc comment still says "A PullRequest other than empty, on and off is refused". `merge-in` is now accepted, and the same diff changed the neighbouring `Task` field comment and the error text.
  - Evidence: the diff hunk at `task.go` 272-306.
  - Fix: reword it to "other than empty, on, off and PullRequestMergeIn".
- **F2, MEDIUM, design.**
  - Location: `internal/landingshed/finalize_pr.go:221-223` and `:312-321` (`recordedBeamLanding`).
  - Summary: `landing.trueMerge` is set only on the non-beam path of `pullRequestLanding`.
    - In `pullRequestLanding`, a beam's landing sets `field=""` and `resolveWithParentRule` forces `entryField=""` for a beam. So a merge-in entry never reaches `TrueMerge` on the beam path. That is correct, because a beam is not a merge task.
    - The `found && StateOpen` early return at line 208 returns before the resolver, so a merge-in entry with a leftover open PR record merges on GitHub as a squash (`atOnce`) instead of a true merge.
    - `landingForDecision` for `Open=false` and `found` drops the record, and `trueMerge` is then set afterwards, which is fine.
  - Net effect: only the leftover open pull request path (line 208) loses `TrueMerge`. That path is reachable if PR-Review or Finalize opened a pull request before the entry became merge-in. Merge-in entries are filed with the field, so this is unlikely. If you treat it as unreachable, downgrade it to LOW.
  - Fix: either set `trueMerge` there too, or state in the `pullRequestLanding` doc that an open recorded pull request wins.
- **F3, LOW, consistency.**
  - Location: `internal/beamcli/mergeparent.go:126`.
  - Summary: the body renders "Filed by`lym beam merge-parent X`" with no space before the backtick. The text is malformed, and the test (`mergeparent_test.go`) asserts only the `merge-in` and `reviews the merge alone` substrings, so nothing pins it.
  - Fix: restore the space (``Filed by `+"`…`"``).
- **F4, NIT, scope.**
  - Location: `internal/boardcli/cli.go:547`.
  - Summary: the `run_settings` help line was reworded ("for the pair's local config" dropped) to fit the Long cap. That is unrelated to the task and loses information. Shorten only the `pull_request` line instead, if the cap allows.

Rules and commits:
- **Declared PATTERN changes:** none declared in the directive. `PATTERN-landing.md` carries a new line, and `pattern/` has no other entry that conflicts.
- **Commit trailers:** none of the four commit messages carries a `Pattern-Departure:` trailer, and none is needed.
- **Stencils:** the diff touches none, so the stencil-wording rule has nothing to check.

Tool calls (3):
1. grep for the `pull_request` value lists across `.go`, `.md`, `.yaml` and `.tmpl` files, because my context lacked docs and fixtures outside the diff. The only stale text was the `normalizeTask` comment.
2. Read the `seam_enforcement_test.go` neighbours and the `boardengine` import sites in `landingshed`. Neighbouring entries use end-of-line comments too, so I raise nothing.
3. Grep for old suffix-name references and `merge-in` uses in `internal/darnengine`, `internal/shedrecipe`, `internal/loomcli` and `internal/webster*`. No other reference to the moved names exists.
4. `sed` on `PATTERN-fabric-boundary.md`, plus a grep for enforcement tests over `fabriccli` imports, to check the new `darnengine` import. I found no violation.
