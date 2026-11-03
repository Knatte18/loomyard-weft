```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision (lens: existing-paths)**

- D1 marker/stable outputs vs. review row's second attempt (`internal/shedadapters/burlerreview.go:238-260`, round token `<n>b`, same `roundReviewPath`): **breaks** (F1)
- D1 marker vs. seat start order ("starts the chair first, then the advisors", `internal/seatengine/doc.go:51`): **breaks** — stale marker removed only after the chair is live (F1)
- D1/D6 vs. reed revive after reboot (`internal/reedengine/lifecycle.go:184-196,794`): survives — gated assistant has no done-when list, is relaunched, D6 resume line exits the shell, run ends died, D1 probe marks it
- D2 copy vs. `archiveStaleOutputs` (`internal/shedadapters/burlerreview.go:210`, `archive.go:25`): survives (D2 adds the copy to the set); the `.lym` original is not in that set, so only the engine's pre-start removal clears it (feeds F1)
- D2 recorded assistant choice vs. `readRecordedFans`/probe (`burlerreview.go:201`): survives — recorded beside fans, read on probe
- D5 link `auth.json` vs. Codex's own token-refresh write: **unproven** (F2)
- D5 rendered `config.toml` vs. Codex writing it itself (trust-prompt accept, first-start `[tui]`): survives — the record keeps roots; next render restores them, as D5's bound states
- D5 home rules vs. operator's `~/.codex/rules` allow rules (spike `report.md:288-291`): survives — lym home renders no rules file (Constraints)
- D7 `CreelHooks` false vs. creel's scheduler (`internal/creelengine/deliver.go:34-35,242-247`): **breaks D1's "no channel to it exists"** (F3)
- D12 re-render list vs. a repo with no `CLAUDE.md`: survives (falls back to neither; acceptance criterion covers "else neither")

**Findings**

F1 — MEDIUM — design — D1/D2: stale marker survives into a second attempt.
The review row runs a second attempt of the same round after a died/timed-out first attempt, under the same `roundReviewPath` (`internal/shedadapters/burlerreview.go:238-260`). D1/D2 derive the assistant's report and marker from the review path and round, so attempt 2 reuses attempt 1's marker (reading `stopped`/`died`). seatengine starts the chair before any assistant (`internal/seatengine/doc.go:51`), and D1 clears the stale marker only "before it starts the assistant"; the `.lym` original is not in `archiveStaleOutputs`'s set (`burlerreview.go:210`). Attempt 2's chair can await and read attempt 1's ending, then go on without an assistant that is still running. The record is silent on the attempt token and on clearing before the chair starts.

F2 — MEDIUM — design — D5/D13: `auth.json` symlink survival across a token refresh is untested.
D13's rationale assumes Codex rewrites `auth.json` in a way a symlink survives; the twill spike lists it as untested ("Whether the shared `auth.json` symlink survives a token refresh; none happened in the spike's hour", `docs/research/codex-twill-spike-2026-10-24/report.md:245`), and the bench harness only notes the link "would change the operator's file" (`docs/research/codex-astra-fan-bench-2026-10-27/data/harness/setup-codex-home.sh:3`). If Codex refreshes by temp-file-and-rename, the rename replaces the link in the lym home with a regular file; D5's Prepare "creates any missing link" but does not replace a link that became a file, so the two homes silently diverge (the operator's `~/.codex/auth.json` goes stale while the lym home's copy refreshes). A fact the design rests on is left unresearched.

F3 — LOW — consistency — D1 vs. D7: creel can still type into the assistant's pane.
D1 says "no channel to it exists"; D7 sets `CreelHooks` false, and creel treats every non-hook strand as a pane-mail strand, typing injects/notices and reminders into the pane (`internal/creelengine/deliver.go:34-35,242-247`). Any post addressed to the assistant's strand name is typed into the Codex pane mid-review; the record states no bound for that path.

Tool calls:
- `burlerreview.go` archive set and attempt loop + reed done-when/resume paths: manifest lacked the review row's retry and archive paths.
- `archiveStaleOutputs` body: to see which files a retry archives.
- `auth.json` refresh evidence in `docs/research/codex-*`: to check whether the D13 link was tested against a refresh.
