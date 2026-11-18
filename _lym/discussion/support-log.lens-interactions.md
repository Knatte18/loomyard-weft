```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's decisions that act on the same state, and each acceptance criterion against the decisions it touches. I made no tool calls.

**Pairs checked**
- D2 × D3, the pick against entries and pruning: held.
- D2 × D4, an entry whose name left the list, against D6 and D8: **broken** (F2).
- D2 × D6, a wiped account file against the resume copy: held. The worktree falls back to the default directory, and D6 copies the transcript from the listed account's directory.
- D3 × D7, the switch reusing the pick's steps: **broken** (F5).
- D4 × D7, a pin against the switch: held, since a pinned worktree is skipped.
- D4 × D5, the resolved account against the status line and limit records: **broken** (F3).
- D6 × D7, switching A to B and back to A: **broken** (F1).
- D6 × D8, the copy against deduplication in `LocateSessions`: held.
- D6's live-holder test × the D7 restart: held. The stop ends the process, so its registry entry goes.
- D7 × D7, concurrent limit hits in one worktree: held.
- D7 × D9, the switch in the prime: **broken** (F4).
- D1 × D4, "with none every launch behaves as today" against the scrub: **broken** (F6).
- D1 × D7, a single account or a pin against the switch and the set verb: held.
- D3 × D9, beams picked at creation under a beams pin: held.

Each acceptance criterion agrees with its decisions, apart from the gaps listed below.

**Findings**

**F1 — BLOCKING — D6 × D7: a switch back resumes a stale transcript**
- D6 copies only when the transcript is absent from the resolved directory.
- Take a session that switches A → B (copied to B), runs on B, then switches back to A after A resets.
- A still holds the pre-switch copy, so nothing is copied and the resume silently loses every turn taken on B.
- The fix: copy whenever another known directory holds a newer copy. Compare the files by modification time, or take the most recently written copy among all known directories, the resolved one included.
- Evidence: D6's "transcript is not under the resolved directory's `projects/<project>/`"; D7 lets a worktree switch repeatedly.

**F2 — MEDIUM — D2 × D8 and D6: an account removed from the list strands its transcripts**
- D2 says a dropped name "reads as no entry … and a resume still finds its transcript by D6".
- D8's known directories are only the listed accounts, the `env` value and `~/.claude`.
- So once an account leaves the list, its directory is no longer searched. The resume is refused, and `LocateSessions` and `tools/tokencount` lose its sessions.
- Either D2's claim is false, or D8 should also search the directories named in the stored entries or usage files. At the least, state the loss as a bound.

**F3 — MEDIUM — D4 × D5 and D7: a worktree with no entry, running on a listed directory, never records headroom**
- D4 resolves "no account" for a worktree with no entry and no pin. The session then runs on the `env` value or `~/.claude`, which may be a listed account's `config_dir`.
- D5 gives the status line only when "the resolved account is a listed one", and writes `blocked_until` on "its account".
- So such a session writes no figures and no limit hit. The pick keeps choosing that account, and D7's comparison of "the account the session was launched under" is undefined.
- The fix: map the effective directory back to a listed account by its cleaned directory.

**F4 — MEDIUM — D7 × D9: the switch gives the prime an entry D9 says only the operator sets**
- D9 says the unpinned prime "has no entry until the operator sets one".
- D7's orch path writes an entry for the prime when its orch hits a limit, which moves the prime off "runs as today" for good.
- The record should state that a limit switch is a second way the prime gets an entry, or exclude the prime from the switch.

**F5 — LOW — D3 × D7: a switch does not advance the counter**
- D7 uses only D3's steps 1 and 2, so the switched-to account's count never rises.
- Switches therefore skew the long-run shares D3 promises. The record should state this, or have a switch increment the count.

**F6 — LOW — D1 × D4: "behaves as today" is not quite true with no accounts**
- D1 says that with no accounts listed every launch behaves as today.
- D4's scrub of `CLAUDE_CONFIG_DIR` applies regardless, so an operator's shell value stops reaching panes even in a hub with no accounts. Scope's "Out" bullet states this only for standalone runs.
- The fix: qualify D1's sentence.

No tool calls were made.
