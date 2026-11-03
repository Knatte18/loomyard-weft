# Round 1 fixer-report — discussion

## Changes

- **F1** (BLOCKING), fixed as proposed, the `/tmp` option.
  D2: the assistant writes its report under `/tmp`, in a mode-0700 directory named for a hash of the review path and the attempt token.
  Once the seat gate passes, lym copies the report beside the marker under the anchor's `.lym`, and only then writes the done marker, so the chair reads the kept copy and a reboot that clears `/tmp` loses no done report.
  Dropped the "workspace sandbox covers them" claim.
  Constraints gains the writable-roots and linked-path fact (spike `report.md` lines 10 and 283), with the warp worktree's `.lym` and `_lym` linking into the weft checkout.
  D2's acceptance bullet names the `/tmp` path and the kept copy.
  Support log: the rejected alternative now covers `.lym` as well as `_lym`, and a new one rejects the worktree's own tree, because an untracked file there would reach the fixer's commit.
  Q5 updated.
- **F2** (MEDIUM), fixed as proposed, both options together.
  D1: the marker and outputs key on the review path and the attempt token (`<round>`, then `<round>b`).
  The review row records the token with the round's assistant choice (D2), so probe and resume derive the same paths.
  On a fresh run the table builder removes stale files before the chair starts.
  The D1 and D2 acceptance bullets were updated to match.
- **F3** (MEDIUM), fixed differently.
  Constraints records the `lym heddle list` targets and that the repository has no `CLAUDE.md` or `AGENTS.md`.
  D12 states that no entry targets `codex-file`, that repository-layer entries need the operator's `lym heddle set` and a render, and that the shipped entries stay on `claude-file`.
  The Open risk now names the operator's command for `nicknames`, and the D12 acceptance bullet covers an entry targeting both file targets.
  Not made: adding `codex-file` to the shipped entries, with an acceptance bullet that Codex gets the shipped entries.
  Their text in `internal/heddleengine/shipped.json` directs Claude's Read, Edit and Write tools and the gate, which a Codex assistant does not have.
- **F4** (MEDIUM), fixed as proposed.
  D5 names `screen_reader_detection_done` and the `model_availability_nux` table, citing the twill spike.
  The last Open risk now states only what stays unproven and the backstops.
  The D5 acceptance bullet names both settings.
- **F5** (MEDIUM), fixed as proposed.
  From the `openai/codex` source at tag `rust-v0.164.0`: `FileAuthStorage::save` in `codex-rs/login/src/auth/storage.rs` opens `auth.json` with truncate and write, without a rename, so writes go through the link.
  D13 records this.
  No rename case arises, so D5 gains no replace step.
  D13 also notes that a logout removes the link and Prepare recreates it.
  Its rationale for a hard link now rests on logout and login, not on a refresh, and the support log's rejected alternative was corrected the same way.
- **F6** (MEDIUM), fixed as proposed.
  D6's bound states that automatic trust plus `--dangerously-bypass-hook-trust` runs committed `.codex/` hooks without review and outside the sandbox.
  It is bounded by trusting only worktrees of repositories the hub runs, and by a Prepare Warn naming a committed `.codex/config.toml`, stated in D5.
  The D5 acceptance bullet covers the Warn.
- **F8** (LOW), fixed differently.
  D1's bound now counts the start, its retries, the timeout and one await cap.
  A dead lym process takes the chair's timeout with it, so the chair waits until a resume's probe writes the marker or the chair is stopped.
  D6: a live Codex whose first turn never starts fails its startup, is not retried, and its marker reads `not started`.
  The D6 acceptance bullet covers this.
  Not made: the proposed "until its own review timeout" and "the marker reads timeout".
  The run's deadline is enforced inside the lym process (`internal/shuttleengine/doc.go:75`, `:86`).
  A `RecordsSessionStarts` launch without a session-start line stays not started (`internal/shuttleengine/doc.go:546-547`, `startup_test.go:190`).
  Shuttle relaunches only a start that died (`doc.go:533`).
- **F9** (LOW), fixed as proposed.
  D1 states that the assistant's strand is a pane-mail strand (D7), that nothing in the design posts to it, and that `seat-directive-assistant` tells it to ignore `[creel]` text.
- **F10** (LOW), fixed as proposed.
  D2 states that `burler-template-assist-reviewer` renders the no-parent directive variant and the edit directive with its tool wording from an engine block.
  D11's list of engine-block passages now includes that wording.
- **F11** (LOW), fixed as proposed.
  The skills-sync Open risk was dropped, and the departure is now a clause in D9's rationale.
  Q7 stays resolved, with its text no longer pointing at Open risks.
- The support log gains a Round 1 entry under Review rounds.

## Self-check

- fix-consistency, LOW, D1's directive still said "outputs" where D2 now has a kept copy: fixed as proposed (D1 names the kept copy).
- fix-consistency, LOW, Prepare's `.codex/config.toml` Warn was missing from D5 and its bullet: fixed as proposed (moved into D5 and its acceptance bullet; D6's bound refers to it).
- fix-consistency, LOW, D11's engine-block list lacked the edit directive's wording: fixed as proposed.
- fix-way-forward, LOW, the heddle operator step left out the render an existing worktree needs: fixed as proposed (the Open risk names `lym heddle set nicknames --targets claude-file,codex-file` and `lym heddle render`; D12 notes the next render).
- fix-proposal, MEDIUM, probe and resume would derive attempt `<round>` paths for a live `<round>b` assistant: fixed differently (the review row records the attempt token, and probe and resume derive paths from the record) rather than probing both tokens.
- fix-proposal and fix-claims, MEDIUM, D6's dropped-prompt ending contradicted shuttle's `RecordsSessionStarts` startup rule: fixed as proposed (fails startup, not retried, marker `not started`).
- fix-proposal, LOW, `/tmp` is a tested writable root on Linux only: fixed as proposed (new Open risk for macOS's `/private/tmp` link).
- fix-claims, MEDIUM, D1's dead-process bound named a timeout that dies with the process: fixed as proposed.
- fix-claims, LOW, the attempt tokens `1` and `1b` read as fixed values: fixed as proposed (`<round>`, `<round>b`).

## Disputed

None.

## Deferred

None.
