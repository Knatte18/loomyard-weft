# Fixer report — discussion round 1

F7 and F8 are `decision`-class and went to the parent; they are left out below.

## Changes

- F1 (BLOCKING): fixed as proposed, taking the hook option.
  D6 gains a bullet: `fabricengine` offers a general workspace-content seam, installed process-wide as `InstallPairFolderNamer` is.
  The hook is handed the hub path, the beam slug and the anchor, and returns extra top-level members that the regeneration splices verbatim, naming no key.
  `cmd/lym/main.go` installs the ide module's hook.
  Constraints gain a `PATTERN-fabric-boundary` line; the Notes bullet and the Scope bullet on VS Code tasks name the hook as the `beam start` writer.
  The support log records the post-regeneration writer as a rejected alternative:
  any other regeneration caller would leave a new beam file without `beam start`.
- F2: fixed as proposed. D5 bullet 1 states that the board subtree's `PersistentPreRunE` resolves `open` with `lymcwd.ResolveWithin`,
  and loads the config and board directory from that layout as it does for the other verbs.
- F3: fixed as proposed, then tightened by the self-check (see below).
  D4 bullet 5 defines the kept targets and states that the escape covers double quotes.
  The D4 Review criterion names the `javascript:`, `data:`, `//host` and `/\host` cases.
- F4: fixed as proposed. D6 names the binary: `selfexec.LymBinary` through `ideengine.LymBinary`, falling back to bare `lym`.
  The Notes bullet lists the settled hook inputs.
- F5: fixed as proposed. D6's Bound states the older-binary strip and its recovery.
- F6: fixed as proposed: the `board open` task's presentation is `"reveal": "silent"`, with a D6 criterion.
- F9: fixed as proposed. D6's Bound moves the tracked `tasks.json` case under `alt+shift+9`.
- F10: fixed as proposed. D1's top level carries the README's section names in order, and D4 groups by that list.
- F11: fixed as proposed. D4 bullet 4 links the design file when the data names one.
- F12: fixed as proposed. D5's Bound adds the running-task prompt.
- F13: fixed as proposed. D3 bullet 3 states the plain cherry-pick of an unclaimed path, and the Bound states the downgrade replay.
- F14: fixed as proposed. D1 names the `<`, `>` and `&` escapes.
- F15: fixed as proposed. Scope Out bullet 1 keeps only the excluded items; Open risks bullet 2 carries `b0e9f535` and `6b037083`.

## Self-check

- fix-consistency 1 (MEDIUM), and the same finding from fix-claims 1 (MEDIUM), fix-way-forward F-a (MEDIUM) and fix-proposal's outside note:
  fixed as proposed. The D6 no-hook criterion asserts "no `tasks` key", since a beam file also carries the `lym.generated` mark.
- fix-consistency 2 (LOW): fixed as proposed. The Notes bullet lists the hub path and the beam slug among the hook's inputs.
- fix-consistency 3 (LOW): disputed, premise false.
  The root `PersistentPreRunE` runs the install block under `if !testing.Testing()` (`cmd/lym/main.go:158`), so every non-test lym process installs the hook.
  No production process regenerates without it.
- fix-claims 2 (LOW): fixed as proposed. The D6 Bound pairs each writer with what it restores:
  a regeneration restores `beam start`, and a seed or spawn restores `board open` and the keys.
- fix-way-forward F-b (MEDIUM): fixed as proposed.
  D5's refusal names the anchored directory to run `lym board rerender` from, since that verb keeps `lymcwd.Resolve`. The D5 criterion matches.
- fix-way-forward F-c (LOW), and the same finding from fix-proposal 2 (MEDIUM): fixed.
  The D3 Bound states that the conflict is expected whenever a newer binary has written `board-data.js`.
  The replay restores the tip and the write is skipped as a diverged Bolt with the records-conflict way forward (`fabricengine.Bolt.divergedDetail`).
  The case recurs on each write by the older binary.
- fix-proposal 1 (MEDIUM): fixed as proposed.
  The link check removes tab, CR and LF throughout the target, and refuses a scheme-less target that opens with any two-character mix of `/` and `\`.
- fix-proposal 3 (LOW): fixed as proposed. The `alt+shift+9` sentence adds the window seeded before the upgrade.
- fix-proposal 4 (LOW): fixed differently. The fork's claim that VS Code reveals on exit 1 only is unconfirmed.
  D6 now says the terminal comes forward when the verb fails with exit status 1, which is how the refusal and a failed opener exit (`clihelp.Abort(ctx, 1)`).
  That holds under either reading of VS Code's rule, so the proposed "exit 1 rather than any non-zero exit" wording was not needed.
- fix-proposal 5 (LOW): fixed. D6 states that a hook member named `folders`, `settings` or the generated mark is a programmer error.
  The regeneration returns it and writes nothing.

## Disputed

- fix-consistency 3 (self-check), premise false: `cmd/lym/main.go:158` installs the hooks in every non-test process, so no production regeneration runs without the hook.

## Deferred

None.

## Rulings

None reached this round.
