---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The red-first exemption is narrower than proposed: a `bug` entry only, with no --amends-plan, PATTERN change or --drops-create, since a title alone would let any of those past the closed scope. A Webster plan conflict also names `goto --to Plan-Write` rather than only a note and resume, since the settled-segment guard refuses a goto to Plan-Burler-Review without an amendment record."
  - id: F2
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The red-first ask at contracts/stencils/darn/darn-template-write.md:117 is unchanged: F1 makes `decision add` take that record in a closed scope, so naming a board note there would be wrong."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Round 1 fixer report

## What changed

- **F1.** `decision add` takes a `bug` entry's `No test seam` or `No bug fixed` record in a closed scope (`gateslot.LiftsRedFirst`, `onlyLiftsRedFirst`). The darn stop, plan conflict, reroute cap, dropped Create, parent-review answer and landing escalate refusals name the board-note route.
- **F2.** `shedadapters.SegmentRan` counts a round review, a generation or settle record, or an archived run directory.
- **F4.** The role files and the darn, sampler, arbiter and Code-Review rubric stencils name the closed-scope route.
- **F5.** `TestScopeClosedAt` drives `scopeClosedAt` over real directories.
- **F6.** The recipe switch test pins `scope_closed`, and a Bouncer entry row asserts its wording.
- **F7.** At the hard cap a closed-scope brief names no in-scope ruling. The static line in `bouncer-template-escalation.md` that named `--in-scope` at the cap is reworded too.
- **F8.** Added the missing rows; `recordCircling` is extracted so the verb's routing can be tested.
- **F9.** The shed recipe spec documents `scope_closed`.
- **F10.** The four facts are back in the `decision add` help.
- **F11.** The plan spec asks for a card's seconds only under a budget line.
- **F12.** `plan_tier0_budget_sec` is `CategoryGuard`.
- **F13.** `_lym/landing/summary.md` names the wider blast radius and the older-binary refusal, and describes the behaviour this round changed. It is edited and left for the loop to commit.

- **Comment gate.** After the report was first written, the comment-line check named wrapped or over-long comment lines in `circling_test.go`, `decision.go`, `loomcli/doc.go`, `parentreview.go` and `archive.go`. Each is rewritten as one sentence per line, in its own commit, and `lym loom lint-comments` passes.

`lym gate test` passed after the last change.

## Self-check

- fix-code-correctness 1, LOW, help and docs promise more than the exemption admits: fixed as proposed.
- fix-consistency 1, MEDIUM, landing-halts note route also covers Finalize: fixed as proposed.
- fix-consistency 2, MEDIUM, parent directive lacks the exception: fixed as proposed.
- fix-consistency 3, LOW, overview darn paragraph lacks the closed route: fixed as proposed.
- fix-consistency 4, LOW, "one exception" beside "two things outside the line": fixed as proposed.
- fix-consistency 5, LOW, scope test still says `roundStarted`: fixed as proposed.
- fix-proposal 1, BLOCKING, closed plan conflict names a refused goto: fixed as proposed, naming the plan writer.
- fix-proposal 2, LOW, darn own-entry drop with no parent loops: fixed as proposed.
- fix-proposal 3, LOW, exemption admits any call by title: fixed with my own change, the `bug` label requirement. A bug entry's record text stays unconstrained, as the parent writes it.
- fix-affected-set 1, MEDIUM, parent directive re-records folded calls in a closed scope: fixed as proposed.
- fix-affected-set 2, LOW, headroom way forward lacks the closed route: fixed as proposed.
- fix-affected-set 3, LOW, over-cap appended bullet way forward: disputed, premise false. `Validate` runs the over-cap check (`internal/discussionparser/validate.go:102`), and `AppendDecisions` runs `Validate` and restores the record on a finding, so the refusal is met only by an add the open scope admits.
- fix-affected-set 4, LOW, run-held record refusal names only `decision replace`: fixed as proposed.
- fix-affected-set 5, LOW, arbiter stencil limits the closed rule to Code-Review: fixed as proposed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
