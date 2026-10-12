# Support log — sed-read-ok

## Interview

Autonomous run: no operator answers, so each turn below is the writer's exploration and its own pick.

1. Read the board entry (`sed-read-ok`, merged `webster-fork-shell`) and the related entries `audit-reader-globs`, `test-tmp-cleanup` and `spawn-audit-log`.
2. Searched every non-test file for `sed`: only the edit directive, `editRulePhrases`, the fabric-reference classifier and its docs, and the refusal-spec row mention it; no plugin skill, orch stencil or `CLAUDE.md` says "never sed".
   The orch's "never sed" came from the old `scribe:conversation`, which the operator already changed; nothing in lyx to fix.
3. Read `claudeengine`'s deny table: no deny matches `sed`; autonomous runs and the orch run with `--dangerously-skip-permissions`; interactive runs prompt by default.
   Concluded read-only `sed` is refused nowhere and prompts nowhere in bypass mode (D8).
4. Read the classifier: only `sed -n Np` and `sed -n N,Mp` are reads, and it is consulted only for fabric-referencing commands.
   Picked a bounded widening (D5).
5. Checked whether the edit directive reaches Webster forks: it renders into Merriam's template, and forks inherit Merriam's context, so it does; the fork ignored it.
   Picked enforcement by deny (D2) over a stencil restatement.
6. Read the `PermissionRequest` path from `shuttle-hook-first`: the hook is recorded and folds to `asking`/`ask`, but the fold is shadow-only and `ParseEvents` sees no ask, so a stalled run reads `running` to its deadline.
   Found loom, Webster and burler runners already notify the orch; picked a bounded notice (D6) with a pane check.
7. Read `PATTERN-no-denied-recovery`: its closed list must follow the deny table, so D7 changes the background file and its test.
8. Lens review: all six forks returned.
   - premises, MEDIUM, `.scratch/` ignored only in loomyard: fixed, D1 seeds `.git/info/exclude` through fabric, with an acceptance criterion.
   - premises and pattern-fit, MEDIUM, exemption marker needs "never" directly before `perl -i` too: fixed in D1, D5's spec row and the acceptance criterion.
   - premises, LOW, `editRulePhrases` "`sed`": fixed, becomes "`sed -i`" in D1.
   - premises, LOW, `grep` alternation falsely denied: fixed, named in D2 and D3 bounds.
   - pattern-fit, MEDIUM, deny steers restate the edit rule: fixed, D2's steer points at the rule.
   - pattern-fit, MEDIUM, scratch rule stated in D1 and D3: fixed, D3's notice keeps the literal-path steer only.
   - pattern-fit, LOW, D4 dropped `awk` silently: fixed, `awk` stays a reader.
   - pattern-fit, LOW, Claude hook names in a provider-neutral doc: fixed, D6 docs split by package.
   - scenarios, MEDIUM, a later main-session signal clears a fork's ask: fixed, D6 uses a pane-cleared latch, with an acceptance criterion.
   - scenarios, MEDIUM, shared `NotifiedOffset` can swallow a deferred hold notice: fixed, D6 persists its own offset, with an acceptance criterion.
   - scenarios, LOW, restart with a pending dialog: fixed, latch rebuilt on attach, with an acceptance criterion.
   - scenarios, LOW, trailing `-i` and `--in-place=.bak`: fixed in D2 and acceptance criteria.
   - scenarios, LOW, `1p;`, `//p`, `/a/Ip` unpinned: fixed, rejected in D5 and acceptance criteria.
   - scenarios, LOW, concurrent dialogs: fixed, one notice per latch episode.
   - goal-scope, MEDIUM, Scope Out contradicted D1: fixed.
   - goal-scope, MEDIUM, fork hook firing left as an assumption: fixed, a live llm-tier acceptance criterion settles it; the open risk now names what a failure means.
   - goal-scope, LOW, tmux exclusion unreasoned: fixed in Scope Out.
   - goal-scope, LOW, why the fork ignored the directive: fixed, D2 says the cause is unknown and moot.
   - goal-scope, LOW, notice wording omits elicitation dialogs: fixed, "permission or input dialog".
   - board-fit, MEDIUM, `test-tmp-cleanup` sweep of a live `.scratch/`: fixed, named under Open risks.
   - board-fit, LOW, `audit-reader-globs` `doc.go` gap: fixed in Scope Out.
   - board-fit, LOW, `creel` and `orch-split` reroute notices: fixed, one line in Notes for the plan writer.
   - board-fit, LOW, ready marker possibly stale: rejected, the marker was written after the forks were spawned and after validation; the fork's transcript view predates the write.
   - leanness, LOW, the two `CLAUDE.md` constraint bullets: fixed, cut.
   - leanness, LOW, duplicate D6 doc requirement: fixed by that cut.
   - leanness, LOW, completion-signal hedge: fixed, cut.
   - leanness, LOW, false-deny risk repeats D2 and D3: fixed, cut from Open risks.
   - leanness, LOW, D1 phrase deliberation: fixed, now a decision.
   - leanness, LOW, Scope Out cited D1: fixed.
   - leanness, LOW, assumption and risk restate one fact: fixed, the assumption is gone and the risk stands alone.

## Rejected alternatives

- **A `permissions.allow` rule `Bash(sed -n:*)`** in spawned agents' settings: a prefix rule admits `sed -n` scripts that write with `w`, and no run that prompts needs it.
- **Restating the edit rule in the Webster implementer stencil**: the directive already reaches forks, and a second statement breaks the single-statement check; a deny enforces where text did not.
- **A config key per new deny**, as the python deny has: no target repo needs `sed -i` or a computed `rm`, so a switch is unasked-for.
- **Denying `awk -i inplace`**: rare, and `awk` mostly writes by redirection, which no pattern catches.
- **D6 on session state alone, without the pane read**: after an operator approves a dialog the state stays `asking` until the next signal, so a long turn would raise a false notice.
- **Recording `PostToolUse` to clear the ask**: a line per tool call grows the events file for a rare case the pane read settles.
- **D6 cleared by any later session signal**: a fork's dialog outlives Merriam's own turn ends and turn starts, so the notice would never fire in the incident.
- **D6 sharing `RunState.NotifiedOffset`**: a dialog notice at a later offset would suppress a deferred held-turn-end notice.
- **D6 waiting for the run deadline**: that is today's behavior, the stall the board entry asks to surface.
- **Scratch under `.lyx/` or the Claude Code session scratchpad**: `.lyx/` is lyx-owned state behind `internal/lyxdirs`, and the session scratchpad sits under `/tmp`, which the operator's rules and `test-tmp-cleanup` push away from.
- **Widening the classifier to `$` addresses or `-e` scripts**: `$` would need the word splitter's no-`$` rule relaxed for every segment kind.

## Review rounds

- Round 1 (BLOCKING): all eight findings fixed.
  - F1, BLOCKING, D1's interpreter wording failed the no-denied-recovery scan: D1 keeps "never `python` or `node` in an inline heredoc", and the acceptance criterion names `python`.
  - F2, MEDIUM, D6's latch never cleared after a notice: pane reads go on after the notice until an input box shows, with an acceptance criterion.
  - F3, MEDIUM, restart repeated a sent notice: D6 persists the latest ask of the episode and never rebuilds from one at or before it, with the acceptance criterion extended.
  - F4, MEDIUM, D2's word extent and Bound: words end at a command separator, flag words open with a single dash, and the Bound and acceptance criterion grew.
  - F5, MEDIUM, D7's closed list covers only bare spellings: D7 and the background file say the other spellings stay outside the scan.
  - F6, LOW, positional parameters: D3's pattern widens to a digit, `@` and `*`; the other special parameters join the lets-through list.
  - F7, LOW, the Goal overclaimed: the Goal names the two forms, and an assumption names Claude Code's `rm` trigger set.
  - F8, LOW, `.scratch/` exclude reach: root-anchored `/.scratch/`, with a Bound.

## Question ledger

- Q1 (auto-pick): does any lyx hook, audit or role refuse or prompt on `sed -n`? — No; see D8.
- Q2 (auto-pick): should the classifier admit more read forms? — Yes, regex addresses and `;`-joined prints (D5).
- Q3 (auto-pick): do stencils, the orch role or `CLAUDE.md` carry stale "never sed"? — Only the edit directive's ambiguous wording (D1).
- Q4 (auto-pick): should a `sed` allow rule go into spawned settings? — No (D8).
- Q5 (auto-pick): does the edit directive reach Webster forks? — Yes, through Merriam's context; enforce with a deny (D2).
- Q6 (auto-pick): should a scratch place be named and computed-path `rm` forbidden? — Yes, in the shared directive (D1) and a deny (D3), for every role rather than only the implementer.
- Q7 (auto-pick): what turns a run stuck on a permission dialog into an operator-visible notice? — D6, through the existing orch notifier.
- Q8 (auto-pick): bound for D6? — 60 seconds, a package constant.
- Q9 (auto-pick): does the `PermissionRequest` hook fire for a fork's dialog? — Not verifiable at design time; a live llm-tier acceptance criterion settles it, and Open risks names what a failure means.
- Q10 (auto-pick): where does `.scratch/` get ignored in a target repo? — Fabric's `.git/info/exclude` seeding (D1).
