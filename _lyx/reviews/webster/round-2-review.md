---
verdict: APPROVED
---

Range: `status.json` gives `product.parent` `main`; `_lyx/loom/rework/round-1/record.json` carries `class` `exempt` and `first_card` 11, matching `_lyx/plan/00-overview.md`, so the range is `git log --first-parent --no-merges -p 00b9be707..HEAD`.
That range is one commit, `bde81b4e1`, changing one line of `PATTERN.md`.

Focus verdict on the `PATTERN-shuttle-stop` entry: the lead-in before the semicolon is not unchanged, and the clause after it is not the card's sentence verbatim.
Neither departure is a defect in the diff.
Card 11 asks for the original lead-in (310 characters with the line break) plus a 208-character clause, and with the card's exact sentence the entry line is 409 characters, counted with `wc -m`.
`internal/pattern/check.go:24` caps an entry line at `MaxEntryLineChars = 320`, and `TestOverview_RealFilePassesCheck` runs that check over the real `PATTERN.md`, so the card's text cannot land as written.
That is a defect in the plan, which this gate does not judge.
The committed line is 316 characters including the line break, so it fits the cap and keeps the card's intent: the stop verb stays the only route, and `loomcli`'s removal of the loom driver's strand and `orchcli`'s removal of the orch strand are the two named exceptions, with the reason that no attach probe or Wait reads their records.
The ordering "settles the run's record first" survives the shortening of the lead-in.

Other checks that held:

- The entry's claim matches the code: `orchcli` removes the orch strand through `RemoveStrand` at `internal/orchcli/stop.go:41` and `internal/orchcli/start.go:225`.
- The card has no Rename, Delete or other typed groups, so there is no per-card mechanical check beyond the plan's `verify:`; `go test ./internal/pattern/...` passes.
- The diff adds no code and no test, so the test-economy and comment-convention dimensions have nothing to flag.

I first drafted two findings against the diff, one for the clause and one for the lead-in, then withdrew both after confirming the round-1 fixer's evidence above: the verbatim text breaks a test-enforced constraint, and the constraint wins.
