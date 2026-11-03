```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision on one concrete case and checked whether it reaches its stated purpose.

**Decisions walked**

- **D1**, the attempt-2 retry of round 2, where the retry's chair awaits the marker for token `2b`: held for a fresh run. A failed seat gate has no marker text (F3).
- **D2**, a probe and resume after lym restarts during attempt `2b`: broken, because the recorded token reads `2`, never `2b` (F1).
- **D2**, a fresh solo assisted round: met. The report is written under `/tmp`, passes review-parse, is copied to `.lym`, then the done marker is written, the chair reads the copy, and Go copies the report beside the review.
- **D3**, `discussion_assist_reviewer_model: ["", "sol[medium]"]`: met. Round 1 runs no assistant and round 2 runs Sol.
- **D4**, the spec `astra[xhigh]` with the default `efforts`: met. It is refused, naming the key.
- **D5**, Prepare in this worktree: met. The `.git` file points to `/home/knatte/Code/loomyard-LYMHUB/loomyard/.git/worktrees/codex-engine`, so the main-checkout root is `.../loomyard`, a real checkout, and both roots are recorded.
- **D6**, a launch with skills, then a reed revive after a reboot: met. The first turn is the skill load and `SessionStart` fires; the resume line exits the shell, so the run ends died.
- **D7**, a `Stop` payload, then an interrupt: met.
- **D8**, the trust prompt, then a paste send: met.
- **D9**, `scribe:prose` under the default root: met.
- **D10**: met.
- **D11**, an engine block missing for Codex: met. The assistant does not start and the marker reads `not started`.
- **D12**, this repository, which has no `CLAUDE.md` or `AGENTS.md` and no `codex-file` entry: purpose not met by default (F2), and its claim about the render is wrong (F4).
- **D13**, a token refresh and then a logout: met.
- **D1 bound**, chair timeout against assistant timeout: held. `review_timeout_min` is 240 and the assistant timeout is 60.

**Findings**

**F1 — MEDIUM, design, D2 and D1 (attempt token in the review row's record)**
The recorded attempt token can never read `2b`, so a probe of an interrupted retry cannot find the retry.
- The record goes "beside its recorded fans", and the fans are written once by `recordRoundFans`, before the attempt loop (`internal/shedadapters/burlerreview.go:229-233`).
- The `b` token is minted inside the loop, at attempt 2 (`burlerreview.go:254-257`).
- `probeLiveReview` hard-sets `opts.Round = strconv.Itoa(round)` (`burlerreview.go:415`), and that Round names the table's run directories (`internal/burlerengine/review.go:58`).
- So a probe after lym dies mid-retry builds paths for `2`, never `2b`. The live chair and assistant of `2b` are not found, and the acceptance criterion "a probe of a retried round finds the retry's assistant" fails.
- D2 needs to say that the record is rewritten with the new token before attempt 2 spawns, and that the probe reads the token from the record for the chair as well as the assistant.

**F2 — MEDIUM, design, D12 (purpose against default end state)**
D12's stated purpose is that "the assistant must get the repository instructions the chair gets", and in this repository it gets none by default.
- The repository has no `CLAUDE.md` or `AGENTS.md`, and no entry targets `codex-file` (`lym heddle list`).
- `fileText` therefore renders no `AGENTS.override.md` (`internal/heddleengine/render.go:250-252`), and the `CLAUDE.md` fallback finds nothing.
- The Claude chair still gets `CLAUDE.local.md`, which holds `lym-batching`, `lym-plan-edits-before-gate` and `nicknames`.
- Parity depends on an operator step the record leaves as an Open risk. This conflicts with the board's rule that a Codex reviewer always gets the context a Claude reviewer of the same round gets (operator, 2026-10-27).

**F3 — LOW, consistency, D1 and D2 (marker on a failed seat gate)**
- When the assistant's review-parse gate exhausts its budget and fails, the run ends done with its gate failed.
- D1's marker texts (`done`, `died`, `timeout`, `stopped`, `not started`, wait error) name no text for that ending. "Passed or let the run through" is not defined either, so it is unclear whether a let-through malformed report gets copied and marked `done`.
- Evidence: `decision-record.md` lines 40-41 and 59; the seatengine contract for a "done with its seat gate failed" ending (`internal/seatengine/doc.go:85`).

**F4 — LOW, consistency, D12 (claim about the render)**
- D12 says "the render carries the anchor's instruction files alone until one does".
- In fact `fileText` returns `false` and renders no file at all when no entry names the target (`internal/heddleengine/render.go:250-252`), so nothing is rendered until an entry targets `codex-file`.

**Tool calls**

1. One Bash call read the worktree's `.git` gitdir and `commondir`, grepped the `review_timeout_min` default in `internal/loomengine/template.yaml`, and grepped how `internal/burlerengine` uses `opts.Round`. My context lacked the main-checkout target behind D5, the chair-timeout default behind D1's bound, and whether Round names the probed table behind D2's probe walk.
