```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: I checked the change set against its sibling cards, the shared decisions and the cross-references.

- **MEDIUM**: the wording at card 5 `05-capture-run.md:74` (StartCapture errors, F10) contradicts card 9 `09-drift-baseline-producer.md:47`.
  - Summary: StartCapture's new way forward ("run `lyx drift capture` again, or `lyx drift defer` followed by `lyx loom resume`") is shared with the row's caller. Card 9 returns that same error from `Drift-Baseline` as a resumable error stop "whose resumed entry captures again". In that state both named verbs refuse.
  - Evidence:
    - Card 5's row refusal "names running `lyx drift capture` only at a `Drift-Await` halt or a `Drift-Baseline` capture halt".
    - Card 7's `Defer` "refuses unless the status is `blocked` at `Drift-Await`, or `blocked` at `Drift-Baseline` with an error opening with `CaptureHaltPrefix`".
    - An error stop at `Drift-Baseline` is neither of those, so the row's halt names two ways that refuse, and never `lyx loom resume`, the one that progresses.
  - Possible fix: word the error by caller (the row: `lyx loom resume`), or key it on the told status as F5 does for `CheckMainCapture`.

- **MEDIUM**: card 11 step 7 (F3) has a twin at step 10 that was left as it was.
  - Summary: step 7 now runs the verify gate only when the last history Stuck's reason "opens with one of those two halts' opening texts (package constants)". Step 10's Stuck after the dirty or verify entry spent its attempts is never said to open with those constants. Step 10 still says its resume "re-runs step 7".
  - Evidence: step 10 "keeps the gate's findings file and names its way forward … then `lyx loom resume`, whose entry re-runs step 7". Step 7's halts are only "a dirty tracked tree is Stuck …" and "A failure is Stuck naming its findings file …". The main path, an agent's spent verify entry and then resume, therefore reads as an unrecognised entry, and no verify gate runs before the compare.

- **LOW**: card 11 step 7, "A dirty tracked tree is Stuck … on every entry".
  - Summary: this contradicts the new step 2, under which a live chair "skips steps 3 to 8".
  - Evidence: step 2's text "A live chair skips steps 3 to 8 and goes straight to step 9"; the dirty check is part of step 7.

- **LOW**: the overview Shared Decision "run status decides the caller" disagrees with card 11 step 2.
  - Summary: the rationale says the mark is "closed at the row's entry and at the agent's end". Step 2 closes it only when it is closed already or no live chair is found, and a live-chair reattach keeps it open.
  - Evidence: `00-overview.md` rationale, against card 11 step 2: "A closed mark, or a probe finding no live chair, closes the mark through `CloseAgentSession`".

- **LOW**: card 7's audience lead-in, "Verbs, with audience `role` for an agent's and `operator` for the parent's:", is stale after F12.
  - Summary: `capture` is now `operator` while its `--side` form is the agent's. The added sentence explains this, but the lead-in still states the old rule.
  - Evidence: card 7, the verbs list: "`capture [--side]` (operator; with `--side`: …".

Tool calls:
- One grep over `_lyx/plan` cards for twins of the changed wording (removing/cleaning, `_lyx/drift` commits, the moved Extends refs, `running`-only caller tests). My context showed the cards only as the fixer edited them, not each card's remaining unchanged text side by side.
