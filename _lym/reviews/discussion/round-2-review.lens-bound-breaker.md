```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break every bound, guarantee and claim about existing code in the record. Four claims break and two hold only partly.

**Claims attacked**
- D1, a tracked-layer value is refused, as shown by `configengine` layer provenance: **broken** (F5).
- D1, an invalid list fails the launch: held.
- D2, a record is kept while its directory exists: **broken** in part, because a directory that exists but is logged out keeps the worktree there (F4).
- D2, transcripts left by an earlier worktree at the same path are not searched: held. Reed's `reed.json` sits under the worktree anchor's `.lym`, so it goes with the worktree.
- D2/D6, the no-record fallback "the `env` module's value, then the process's, then the default, as Claude Code itself would for a pane": **broken** (F2).
- D2, the concurrent-writer re-read under one lock: held.
- D3, a new account "does not catch up on past picks": **broken** for a re-added account or a raised weight (F3).
- D3, seeding at the lowest count over weight: holds, but is undefined when no listed account has a count (F6).
- D3, the fallback ranges over candidates only: held.
- D4, the export comes after the `env` exports and the relaunch replays the stored command: held. The launch command is the last part of the pane line, and `buildLaunchCmd`/`buildOpenCmd` carry the whole line.
- D4/D1 AC, with accounts empty and no record, an `env` module `CLAUDE_CONFIG_DIR` "stands as today": **broken** (F2).
- D5, a forged input is bounded to one window until the forged reset: held. A later reset wins the merge and an earlier real one is dropped, so the forgery holds until its own reset, as stated.
- D5, an idle session repeating old figures never lowers them: held.
- D6, the record readers read every known directory, including the listed accounts' directories and the hub's worktree records: **broken** (F1).
- D6, "the `env` module's value reaches only panes, gate tiers and Drift steps, never lym's own process": **broken** for any lym process started inside a pane (F2).

**Findings**

**F1 — BLOCKING — design — D6 (decision-record.md:200–206), AC lines 302–303**
- **Summary:** The record readers cannot know the listed accounts or the hub's worktree records. `lym report`'s landed-run criterion cannot be met as designed.
- **Counterexample:** A run lands on listed account B; its worktree and record are gone. `tools/tokencount` and the usage recorder get their engine from `shuttleengines.RecordReaders()`, which builds every engine with `configengine.Dirs{}` (internal/shuttleengines/shuttleengines.go:222–228; tools/tokencount/main.go:341; internal/loomcli/wiring.go:640 and looparm.go:140). The machine layer stacks only when `Dirs` carries both `Anchor` and `Board` (internal/configengine/doc.go, machine-layer paragraph). So `claude.accounts` and the `env` module load empty, the known directories shrink to the process's `CLAUDE_CONFIG_DIR` and the default, and B's sessions are not counted.
- **Second gap:** Listing "the directories of the hub's worktree records" means enumerating the hub's worktrees. An engine is told its paths and derives none (PATTERN-cwd-resolution, Told geometry), and `LocateSessions(worktree)` is told one path only.
- **Record's position:** Silent on how a reader gets the hub geometry.

**F2 — MEDIUM — design — D2 (line 79), D4 (lines 135–136), D6 (lines 205, 211), AC line 272**
- **Summary:** An account's `CLAUDE_CONFIG_DIR` leaks into the hub's tmux server, so a pane the launch exports nothing for runs under that account rather than under the `env` module's value, the process's value or the default.
- **Counterexample:** Once D4 exports `CLAUDE_CONFIG_DIR=B` in a session's line, every lym process run from that pane inherits it: agent Bash, hooks, the orch's verbs. Reed spawns a tmux server from `os.Environ()` minus the claude scrub (internal/reedengine/lifecycle.go:456). That scrub strips only `CLAUDECODE` and `CLAUDE_CODE_*` (internal/shuttleengine/claudeengine/scrub.go:9). A server restarted from such a process keeps `CLAUDE_CONFIG_DIR=B` for every pane after it.
- **Where it bites:** In three paths the launch exports nothing:
  - accounts empty and no record;
  - a resume with no record whose transcript is found nowhere;
  - after the list is emptied.

  In each of them the pane runs under B. That breaks the following:
  - D1's AC claim that the `env` value "stands as today";
  - D6's "as Claude Code itself would for a pane" (the pane reads the server's environment, not the reading process's);
  - D6's claim that the `env` value "never" reaches lym's own process.
- **Fix:** Add `CLAUDE_CONFIG_DIR` to the claude engine's scrub, or have the launch always set or unset it when the list is non-empty. Then state the bound.

**F3 — MEDIUM — design — D3 (lines 120–121, 126)**
- **Summary:** Counts kept for unlisted names, and weights that change without rescaling, give a catch-up burst, which the record says does not happen.
- **Counterexample, re-added account:**
  1. A and B, weight 1 each, 100 picks each.
  2. B is removed, and A takes 100 more picks (A=200).
  3. B is re-added. The counter still holds B=100, so B is not seeded, its ratio 100 is below A's 200, and B takes every one of the next 100 eligible picks.
- **Counterexample, weight change:** Raising B's weight from 1 to 2 halves B's count over weight in the same way, and B takes picks in a burst.
- **Contradiction:** "does not catch up on past picks" and "spreads the picks evenly … rather than in bursts" both break.

**F4 — LOW — design — D2 (lines 69–70)**
- **Summary:** "Kept while its directory exists" keeps a worktree on an account whose login was removed. Every fresh session then stops at Claude Code's login prompt, and the record gives no bound or way forward for it.
- **Counterexample:** The operator logs out account B, deleting `.credentials.json`, and removes B from the list. The directory still exists, so the record is kept and exported. D3's credentials check runs only at a pick, and autonomous runs in those worktrees stall at the login prompt.
- **Record's position:** It covers a directory that is gone (lines 81, 93–95) and an expired login at pick time (line 113), but not a directory that is still there and logged out.

**F5 — LOW — consistency — D1 (line 48)**
- **Summary:** The record says the tracked-layer refusal is "told by the layer provenance `configengine` reports for each unit". That provenance names only the highest layer holding a key (`Stack.Origins`, internal/configengine/layers.go:589–620; `Provenance`, layers.go:626–686).
- **Counterexample:** The worktree's tracked `claude.yaml` holds an account list and the machine file holds one too. Origins reports `machine`, and the tracked value goes unrefused.
- **Fix:** Name the check as reading each tracked layer's document (`Stack.Layers`).

**F6 — NIT — consistency — D3 (line 120)**
- **Summary:** The seed is "its weight times the lowest count over weight among the listed accounts that have a count". It is undefined when no listed account has a count.
- **Counterexample:** The counter file holds only names that were dropped or renamed.

**Tool calls**
1. A grep of `internal/reedengine` for the scrub and environment at server spawn, of the tokencount and report reader construction, and of `reed.json` storage. My context had the scrub keys but not where reed applies them, not how tokencount builds its `ReadSet`, and not where `reed.json` lives, which I needed to test the earlier-worktree claim.
