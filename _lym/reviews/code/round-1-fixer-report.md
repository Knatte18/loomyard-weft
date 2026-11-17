# Round 1 fixer-report

## Changes

- **F1 (MEDIUM), fixed differently.**
  Commit `7952d9fb`.
  `FixHold`'s refusal for a hold not standing as written after the push (`internal/boardengine/holdwrite.go`) now builds its way forward from `holdSubjectNames(want.Subjects)`, the hold's own subjects.
  The FixHold lost-hold test row now holds `beamer boxed`, fails `boxed`, and asserts the way forward names `lym adopt beamer boxed`.
  Not made: the proposed change at the no-hold-standing refusal (line 403).
  No hold exists there, so `want` is still the zero `Hold` and `want.Subjects` would give a bare `run \`lym adopt \``;
  with no hold standing, a re-run over any set is accepted, so the keys of `tries` stay right.
- **F2 (LOW), fixed as proposed.**
  Commit `dec76550`.
  `rewriteOverOrigin` (`internal/fabricengine/takeoverwrite.go`) splits the check: a non-nil `err` keeps the `%w` wrap, and `!hasUpstream` returns an error naming the branch as having no upstream, both joined with `b.dropEntry(rec)`.
- **F3 (NIT), fixed as proposed.**
  Commit `d9e441b3`.
  The `Long` text of `Command` (`internal/adoptcli/cli.go`) says a plain adopt refuses while another machine holds the hub, naming that machine and its time.

- **Gate finding, fixed.**
  `TestCLIForm_EveryCommand` failed: the F3 wording pushed the `lym adopt` `Long` text to 2023 bytes, over the 2000-byte cap.
  The `--take-hold` sentence in `internal/adoptcli/cli.go` is shortened ("naming it and its time", "moves that hold here"), and `lym gate test ./cmd/lym ./internal/adoptcli` passes at tier 0.

Earlier gate: `lym gate test` over `./internal/boardengine ./internal/fabricengine ./internal/adoptcli` passed at tier 0, and `--tier 1 ./internal/boardengine` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings; it confirmed the line-403 departure from F1's `fix` key.
- fix-affected-set: no findings; it confirmed no other `adoptAgainWayForward` site, `%w`-over-nil branch or "waits on" wording remains.

## Disputed

None.

## Deferred

None.

## Rulings

None.
