# Round 2 fixer-report — discussion

All changes are in `_lym/discussion/decision-record.md`.
`_lym/discussion/support-log.md` is unchanged.

## Changes

- **F1** (MEDIUM): fixed differently.
  D2's shape-check sentence is replaced by three:
  - a non-mapping `buttons` key fails the load;
  - a group of any other shape, a mapping included, loads in reed's hub base and worktree files and desktop's hub file and is skipped at render with the invalid-entry Warn, as is an empty mapping in desktop's machine layer;
  - a non-empty mapping group in desktop's machine layer fails desktop's load, adds no machine buttons and fails `afterWindow`.

  The D2 criterion now names a mapping group in `reed.yaml` that loads and is skipped at render.
  Two parts of the proposal were not carried out, since the self-check showed them false:
  - "with configengine's unknown-keys Warn when it is a mapping": `pruneMapping` (`internal/yamlengine/layers.go:396-425`) records no removed key when a mapping meets a sequence template, so no such Warn fires.
    The text says "with no load-time Warn".
  - the machine-layer refusal covers every mapping group: `unitPaths` (`internal/yamlengine/layers.go:505-509`) does not descend an empty mapping, so `ValidateMachine` admits it.
    The refusal now names a non-empty mapping.
- **F2** (MEDIUM): fixed as proposed.
  D7's click-line bullet now says the pinned line, click and confirmed alike, ends with a forced zero status in reed's pane shell dialect.
  The "discarding the output alone" sentence is replaced: the discarded output and the forced zero status together keep view mode shut, whatever the binary or verb does.
  The verb's exit-0 rule stays.
- **F3** (MEDIUM): fixed as proposed.
  `## Auto-mode assumptions` gains the line saying that a task session's commands run in its beam's anchor, or the prime's when the beam is absent, not in the task worktree, and that `{worktree}` names the task.
- **F4** (LOW): fixed differently.
  The false Windows rationale is deleted from D7's run-directory bullet.
  The proposal restated the `ResolveBattenPrime` reason in D7, but D8's rationale already states it, so D7 points to D8 instead ("so the default CANCEL needs no `cd` (D8)").
  This follows the say-it-once rule, raised by the fix-proposal lens.
- **F5** (LOW): fixed as proposed.
  D7's identity clause is narrowed to a direct call that runs the entry itself.
  The Bound gains the sentence on the confirm path, where a `y` runs the entry as the operator from the tmux server.

## Self-check

Each line gives the lens, the severity and the disposition.

- fix-claims F1, MEDIUM, merged with fix-proposal 1: fixed. The unknown-keys Warn clause is replaced with "no load-time Warn".
- fix-claims F2, LOW: fixed. The machine-layer refusal names a non-empty mapping, and an empty mapping joins the loads-and-skipped sentence.
- fix-consistency 1, MEDIUM, merged with fix-proposal 2: fixed. D2's render sentence now says "a type error the layers admit" leaves both loads whole.
- fix-consistency 2, LOW: fixed. D2's rationale "costs only itself" gains the machine-layer non-empty-mapping exception.
- fix-proposal 3, LOW: fixed. D7 points to D8 instead of restating the `ResolveBattenPrime` reason. The Auto-mode line stays, since F3 requires it there.
- fix-proposal 4, LOW: fixed. D7's Bound gains the sentence that a click whose pinned binary is gone runs, logs and shows nothing until a reed op of another build, not older than the pinned one, re-pins the bindings (`bindingsStale`, `internal/reedengine/bindings.go:54-71`).
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
