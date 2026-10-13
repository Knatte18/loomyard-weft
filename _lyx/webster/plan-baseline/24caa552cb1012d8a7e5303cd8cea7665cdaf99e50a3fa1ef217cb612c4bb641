# Card 10 — restore-set-content-rows

**Create:**
- `plan:internal/reedengine#Engine.SetContentRows` -> `func (e *Engine) SetContentRows(name, paneID string, rows int) error`
- `plan:internal/reedengine#TestSetContentRows` -> `func TestSetContentRows(t *testing.T)`

**Edit:**
- `internal/reedengine#applyOpts`
- `internal/reedengine#Engine.applyLayoutLockedOpts`
- `internal/reedengine#TestZoomBracket_ZoomedStrandSurvivesEveryReedStep`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Restore content sizing's reporting operation, which commit `4ddcb5133` deleted, so an explicit content-sized caller can still report its line count.
The decision record's Scope Out keeps content sizing valid for an explicit caller;
D7 drops only the two overview panes' calls of `SetContentRows`, not the operation.
The battencli package-doc fix (`c7fc40966`) and the `SkipWhenBoxEquals` removal (`17dc83f41`) stay as they are.

The restored code is what `4ddcb5133` removed, read with `git show 4ddcb5133`, applied against the current tip:

- `Engine.SetContentRows` goes back into `internal/reedengine/strand.go` after `Engine.UpdateStrand`, with its body unchanged and its doc comment unchanged but for its last clause, "and the pane process retries at its next poll after any refusal", which is dropped, since no pane process reports or retries any more, so the comment's last sentence ends after "so it can resize no strand": it refuses `rows < 1` before taking any lock, then under the op lock and the zoom bracket refuses an unknown name, a strand that is not content-sized and a strand whose recorded pane is not `paneID`, each naming the strand and the reason;
  a count equal to the recorded one touches neither `reed.json` nor tmux, and a new count is persisted and applied with `applyOpts{SkipFocus: true, InstallPins: true}`.
  Bound: the name and pane checks keep a hand-run or replaced pane process from resizing any strand, and the operation changes nothing but that strand's `ContentRows` and the layout and pins derived from it.
- `applyOpts` regains its `InstallPins bool` field and comment, beside `SkipFocus` only, since `SkipWhenBoxEquals` stays removed.
- `applyLayoutLockedOpts` returns early on `opts.SkipFocus && !opts.InstallPins` and skips the `select-pane` on `opts.SkipFocus || focus == ""`, so a `SkipFocus` apply with `InstallPins` installs the window-resized hook array but never moves focus.
  Its doc comment's sentence that only the focusing path installs the hook array names the `InstallPins` exception.
- `TestSetContentRows` goes back to the end of `internal/reedengine/strand_test.go` as it was, its rows unchanged: a count below 1 refused before any tmux call, an unknown name, a positional strand and a report from another pane each refused with the count and state unchanged and no layout applied, a new count persisted and applied with a pin and the hook rebuilt and no `select-pane`, and the same count again touching nothing.
- `TestZoomBracket_ZoomedStrandSurvivesEveryReedStep` in `internal/reedengine/zoom_integration_test.go` regains its content-strand part: after the resume it adds a top content strand and reports 5 rows with the child zoomed, then with Selvage zoomed reports 6 rows, unzooms and asserts the content pane's height is 6.
  The "AddStrand with Selvage zoomed" step `4ddcb5133` put in its place goes, the `render` import `8422f9434` dropped comes back, and the doc comment says the content-rows reports again.
- In `internal/reedengine/doc.go`, `SetContentRows` goes back into the zoom-bracket call-site list after `UpdateStrand`, and its paragraph comes back in place of the line "No pane process reports a content line count today, so a content strand's ContentRows stays unreported and the strand takes its minimum rows."
  The restored paragraph's Bound sentence ends after "so it can resize no strand", dropping "the pane process retries at its next poll after any refusal, which is how a report made before AddStrand has recorded the pane succeeds once it has".
  The restored paragraph gains one sentence saying neither overview pane reports a count, so the operation stays for an explicit content-sized caller.
- The `internal/reedengine` row in `cmd/lyx/testdata/test-budget.yaml` rises by one, from 264 to 265, for `TestSetContentRows`.

**ImpactSummary:** Restores one exported reedengine method, one unexported apply option and their tests and docs; no caller outside `internal/reedengine` changes, and the overview panes still never call `SetContentRows`.

**Commit:** 10: restore-set-content-rows

**Verify:**
go vet -tags tmux ./internal/reedengine
