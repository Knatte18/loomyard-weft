```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Walkthrough list**
- **D1:** `run_settings` sets `discussion_assist_reviewer_model: fable[high]` → `ResolveReview` → `resolveAssistModels` → assistant seat in the round. Purpose met.
- **D2, goto case:** round k asks, the parent answers satisfied, and the Bouncer escalates round k. Then `goto --to Discussion-Write`, which appends a `Discussion-Write` goto entry stamped now (`goto.go:140`). Discussion-Write reruns, the Bouncer halts Awaiting on round k again (no clear on an undecided escalated round, `bouncer.go:367-369`), and `continue` starts round k+1. The seed is the same, and the history entry is later than the latest ask, so round k+1 asks. Purpose met.
- **D2, clear case:** `entryClear` → `archiveRunDir` (records carried, `archive.go`) → `seedCall` writes a new round-1 focus file with a new write time, so the seed differs and round 1 asks. Purpose met.
- **D3:** the row resolves the strand, writes `pending-post`, posts signed as LOOM, then sets `open`. With no orch strand recorded, `notifyPrime` returns `false, nil` (`notify.go:193-195`) and the ask goes `no-parent`. Purpose met.
- **D4:** the orch's fork carries the orch's strand env and runs `answer <slug> --ask n --findings-file f`. The guard passes on `strandName == parent.Name` (`decision.go:693`), and the ask goes `done`. Purpose met.
- **D5, chair-consolidates case (timeout / `not-sent`):** the chair runs `await` and gets `ready: true`, then has to write "the parent review did not arrive and why". Purpose not met: no step hands the chair the ask's state or reason (F1).
- **D6:** a fork in the orch session shares `AGENTNAME` and passes D4's guard. Purpose met, with the bound as the record states it.
- **D7, chair skips or misreads the await case:** the parent answers `done` with findings before the table ends, and the report is copied beside the review. The review holds none of its findings, and the Bouncer approves. Purpose not met (F2). The merge case breaks too: a parent finding merged into a duplicate the chair kept may carry `origin: handler` (F2).

**Findings**

F1 — MEDIUM, design — D5 and D7: the inputs the chair consolidates from are not defined where the verbs hand them over.
- D5 defines the await's answer only as `ready: true|false` (`decision-record.md:155-157`). Yet D5 has the chair write a line naming why the parent review did not arrive (`:167`), and the criteria require that line to name `timeout` or the `not-sent` reason (`:244-245`).
- D7 has the chair "read a `done` parent report" (`:194`). D4 puts that report "beside the ask record" under `.lym`, keyed per run and not per review attempt (`:116`, `:91`), and no decision says how the chair's prompt or the await's answer names its path.
- For comparison, the assistant's report and end marker are seat outputs whose paths the chair's prompt names at compose time (`assist.go:119`, `burler-assist-consolidation.md:9`).
- Fix: D5 states the await's answer fields (the ask's state, the `not-sent` reason, and the report path when `done`), or D7 says the parent consolidation part is filled with the report path.

F2 — MEDIUM, design — D7: "keeps every parent finding" rests on chair prose alone, and its criterion asks a test to check an LLM's output.
- The chair can consolidate without running the await. D5 then times the ask out only once the table ends (`decision-record.md:159-160`). An answer the parent made before that point is `done` and copied beside the review (D4, `:131`), but the review holds none of its findings.
- When the Bouncer approves that round, those findings never reach Plan. D2's re-ask needs a later round to run (`:72-73`, `:80`).
- D7 also lets the chair merge a parent finding with a duplicate of its own (`:196`) without saying the merged finding keeps `origin: parent`. The D7 criterion requires each parent finding to carry `origin: parent` (`:250`).
- Nothing mechanical checks any of this. The review gate is the existing parse gate (`burlerengine/review.go:271-276`).
- Fix: either add a Go check after the table, refusing or re-prompting a review that lacks an `origin: parent` finding per item of a `done` report (and stating that a merged finding keeps `origin: parent`), or mark the D7 criterion `Review:` and state the bound as instruction-only.

Tool calls:
1. Grepped `wiring.go` for where `segmentAssistBounds` is called, and read `burler-assist-consolidation.md` lines 1-30. My context lacked the consolidation stencil's text, which shows how the chair learns the assistant's report and marker.
