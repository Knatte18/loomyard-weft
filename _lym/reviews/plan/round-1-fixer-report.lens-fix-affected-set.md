```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: reran the set searches behind F1, F7, F9 and F11 to check that the fix covers every member, not only the ones the review named.

Unchecked: F2 (record writes; card 9's pr-continue already goes through `UpdateRecord`, and I did not search code because the record is new), F5 (host shapes) and F10 (one entry), since neither of the last two rests on a set.

F1 is fully covered. The `//lym:seam internal/landingshed#Deps` testers (`wiring_beam_test.go`, `wiring_seams_test.go`, `wiring_darn_test.go`, `landingdeps_test.go`, `battencli/wire_seams_test.go` and envkit ×3) are all named in card 11. The only `Landing.*` nil maps are the two `intentionallyNil` maps, both now targeted.

F11 is covered. Outside `internal/hubissues`, only `selfreportengine` calls `.Issues.Create(`, which the guard does not scan for and which is not a close.

Findings:

- MEDIUM — card 11's new pawl-sequence sentence, "`internal/loomrecipe/doc.go`'s pawl sequence reads … Beam-Describe → PR-Review → Finalize …".
  The F9 set misses one member: `internal/loomcli/doc.go:11` also lists the pawl rows: "Its shed reads Beam-Up, Beam-Await, Parent-Align, the Code-Review perch, Beam-Describe, Finalize and Beam-Down".
  Card 11 adds PR-Review to the pawl recipe but keeps `internal/loomcli/doc.go` under Uses and never rewrites this line, so the line goes stale.
- MEDIUM — card 4's new sentence on gate-key docs (gatecli example, `PATTERN-test-speed`, `PATTERN-gate-slot`).
  The F9 key-list set misses `docs/dotnet-hub.md:88-100`, whose "Gate keys" `gates:` block lists every key lym reads, `drift-verify` included.
  It gets no `pr-review` line, and card 4 does not target the file.
- LOW — card 4, same set.
  `internal/loomshed/doc.go:46` lists each fix row's own gate key ("`code-burler-fix` … `darn-write` … `drift-verify` …, so each fix row and writer runs its own key"), and the change adds no `pr-review` there.
  This is weaker, because that sentence is about the loomshed rows' commands, not every key, so it may be out of scope.
- LOW — card 5's rewrite of the empty-`OutputFiles` precondition.
  The F7 set leaves `internal/shuttleengine/doc.go:186` stale: "one still reading running whose output files all exist keeps running, so a stop never turns finished work into a stopped record".
  Card 5's `stopOutcome` change makes an empty list read stopped, so that sentence no longer holds for an unawaited run.
  Card 5 only says the package doc "gains a paragraph".
  `spec.go:147-168`, the validate doc saying "OutputFiles must name at least one", is in card 5's Edit targets, but the card's Intent never says the comment changes.

Tool calls:
1. Grepped for `RerouteReviewed` across Go files to find every `Landing` nil-seam list (F1); context held only the two maps.
2. Grepped the empty-`OutputFiles` precondition wording across shuttleengine, `pattern/` and `PATTERN.md` (F7) to find restatements beyond those seen.
3. Grepped shuttleengine `doc.go`, `PATTERN-shuttle-provider-seam` and `spec.go` for output-file claims (F7) not shown in context.
4. Listed files naming `drift-verify` and the pawl row sequence (F9) to find every gate-key list and pawl-row list.
5. Grepped the `drift-verify` lines in candidate files and the `.Issues.*` mutating calls (F9, F11).
6. Read `docs/dotnet-hub.md:80-100` to confirm its block is a complete gate-key list.
