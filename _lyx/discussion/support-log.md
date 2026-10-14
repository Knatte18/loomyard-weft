# Support log — hibernate-trigger

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry and the rest of the board.
   `orch-split` and `reed-slack-relay` touch how the operator reaches the orch from the phone, and neither covers a hibernate; nothing on the board duplicates this task.
   The orch hibernated the machine ad hoc on 2026-10-13, which the board's direction that no LLM session runs the hibernate answers, and which the sleep deny (Decision 10) guards against.
2. Explored `internal/tenderengine`, `internal/tendercli`, `internal/pauserecord`, `internal/battenshed`'s doc, `internal/battenrecipe`'s row names, `internal/orchengine`'s doc, `internal/shortcutcli/shortcuts.yaml` and the standing deny table in `internal/shuttleengine/claudeengine/settings.go`.
   Findings: the pause record is hub-scoped; tender's pause already waits out busy sessions and network operations and reports `still_busy` and refused sessions; `SleepWatch` already detects machine sleeps; there is one hub on this machine and no machine-level hub registry.
3. Settled the executor: a detached daemon spawned by `request`, not batten's Land row (Decision 5).
4. Settled the firing sequence, its confirmation rule and its bounded retry (Decision 6), and the wake handling (Decision 7).
5. Settled cancel, callers, platform and the sleep deny (Decisions 3, 8, 9, 10), and the shortcuts (Decision 12).
6. Lens review by six forks; every fork returned a report.
   - premises, BLOCKING: a held parent lock is normal under a pause — fixed: confirmation keys on batten's land lock instead (D6).
   - premises + scenarios, BLOCKING: pause entries have no identity and a pre-existing hub entry leaves the daemon owning nothing — fixed: caller label `hibernate`, owned entry recorded by scope, caller and paused-at, lift by identity, none owned over a live hub pause (D7).
   - scenarios, BLOCKING: daemon death leaves the hub paused forever — fixed: `cancel` of a dead daemon's request works in any phase and lifts its entry; a replacing `request` lifts it first (D4, D8); cold boot named as an open risk.
   - goal-scope BLOCKING + premises + scenarios MEDIUM: the orch is outside confirmation — fixed: the orch must read idle (D6).
   - premises, MEDIUM: background work unaccounted — fixed: a session with background work withholds confirmation (D6).
   - premises + scenarios, MEDIUM: an operator resume mid-sequence goes unseen — fixed: the daemon checks its entry is live before the hibernate call and ends `cancelled` (D6, D7).
   - premises LOW: status does not return `still_busy`/refused — fixed: the re-read is rebuilt from status session states (D6).
   - goal-scope MEDIUM + premises + scenarios + pattern-fit: `--after` on a done-only basis misses a teardown halt, invalid slug waits forever, missing status undefined — fixed: fire once Land is finished, refuse a slug with no batten status, fire at once when already past Land, a vanishing status file ends `failed` (D2, D5).
   - scenarios MEDIUM: racing requests — fixed: check, write and pid record under one lock hold (D4).
   - scenarios LOW: `--after-all` with no run — fixed: fires at once (D2).
   - scenarios LOW: cancel racing the pause — fixed: bound added (D8).
   - goal-scope MEDIUM: hub scope departs from "on the machine" silently — fixed: D4 states the departure.
   - goal-scope LOW: deny not named by the entry — fixed: flagged in Scope as an addition.
   - goal-scope LOW: shortcuts reserved for the operator — fixed: D12 says so.
   - goal-scope + leanness LOW: D7's conditional lift — fixed: decided as lift by identity.
   - goal-scope LOW: resume on wake unrequested — rejected: already an auto-mode assumption, visible to parent review.
   - pattern-fit MEDIUM: retry capped by time only — fixed: at most 120 re-reads (D6).
   - pattern-fit MEDIUM: batten status paths' owner unnamed — fixed: told from their owning declaration (D5).
   - pattern-fit MEDIUM: daemon fires from a replaced binary — fixed: named as an open risk.
   - pattern-fit LOW: `systemctl` teardown logging — fixed: spawn and exit logged (D6).
   - pattern-fit LOW: steer points pair agents at a refused verb — fixed: steer names asking the parent (D10).
   - pattern-fit LOW: cliwire bypass unstated — fixed (D1).
   - pattern-fit LOW: caller parse not via `agentname` — fixed (D3).
   - pattern-fit LOW: daemon spawn kind and binary resolution — fixed: the orch watcher's detached self-spawn (D5).
   - board-fit MEDIUM: deny only in the Claude engine — fixed: open risk naming `codex-engine` and `llm-agnostic-engine`; the deny stays under `claudeengine` per `PATTERN-shuttle-provider-seam`.
   - board-fit LOW: `orch-split` could absorb the daemon — rejected: an unbuilt note, nothing to depend on.
   - board-fit LOW: `spawn-audit-log`, `reed-slack-relay` — rejected: no conflict, as the fork said.
   - leanness MEDIUM/LOW: Constraints repeating cited PATTERN entries, "untouched" and docs bullets, verbatim auto-mode assumptions, D1 and D5 deliberation, D6's `proc`/`exec` choice — fixed: trimmed to pointers and moved deliberation to Rejected alternatives.

## Rejected alternatives

- **Batten fires the hibernate after its Land row.** Ties a machine-level side effect to one task's recipe, cannot serve the immediate or all-runs triggers, and a batten run is busy tearing down at that moment.
- **The orch runs `systemctl hibernate` itself.** Leaves the orch's session hanging mid-call; the board's direction rules it out.
- **A machine-level request covering every hub, as the board's "flag on the machine" reads.** No registry of a machine's hubs exists, the pause record is hub-scoped, and the operator runs one hub; building a registry is a requirement nobody asked for.
- **Leave the hub paused after the wake for a manual `lyx resume`.** The pause exists only to make the hibernate safe, and the operator hibernates with runs live, so a lingering pause only stalls runs.
- **Put the verbs under `lyx tender`.** Tender is about pausing sessions; a machine power action with its own daemon and request file is a module of its own that uses tender.
- **Fire only when the batten run reads done.** A teardown halt after a successful Land would leave the request armed though the task landed; firing once the current row is past Land needs no new batten state either.
- **Confirm on "no parent lock held".** A paused run keeps its parent lock by design, so the pause would never confirm.
- **Lift the hub entry on wake whoever added it.** Would lift an operator's own pause.
- **Config keys for the retry and wake windows.** No requirement asks to tune them.

## Review rounds

- Round 1, BLOCKING; every finding fixed.
  - F1 BLOCKING: confirmation named pause-report categories status never returns and let `unknown` through — fixed: confirmation over the status state names, `busy` and `unknown` withhold, the pause report's lists feed only the first reading (D6).
  - F2 BLOCKING: over a pre-existing hub pause nothing required a live entry — fixed: confirmation and the pre-hibernate check require a live hub-scope entry, owned or not (D6).
  - F3, F4 MEDIUM: `loginctl` has no sleep verbs, `systemctl sleep` undenied, match shape and bypasses unstated — fixed: `systemctl` alone with five verbs, bound states the command-position match and what passes (D10).
  - F5 MEDIUM: `watch` had no caller check and no behavior outside `request` — fixed (D3, D5).
  - F6 LOW: liveness test unnamed — fixed: pid and start time (D4).
  - F7 LOW: pause grace unstated — fixed: ten seconds, fixed in code (D6).
  - F8 LOW: refusals and terminal paths without criteria — fixed: criteria added.
  - F9 LOW: deliberation narrative in the decision record — fixed: moved here or deleted.

## Question ledger

- Q: What does "every instance" mean? A (auto-pick): every session of the hub, as a hub-wide `lyx pause` covers; one hub per machine today. Resolved.
- Q: Who fires the hibernate? A (auto-pick): a detached `lyx hibernate watch` daemon spawned by `request`. Resolved.
- Q: Which triggers? A (auto-pick): now, `--after <slug>`, `--after-all`. Resolved.
- Q: When is the pause confirmed? A (auto-pick): a live hub-scope entry, every session the orch included reading `idle-done`, `idle-stalled`, `idle-fresh`, `asking` or `dead`, no `still_busy` or refused session in the pause report, no network operation in flight, no land lock held; at most 120 five-second re-reads. Resolved.
- Q: Which pause entry does the daemon own? A (auto-pick): the one its pause added, recorded by scope, caller label `hibernate` and paused-at; none over a live hub pause. Resolved.
- Q: Resume after the wake? A (auto-pick): the daemon lifts its own entry and queues an orch notice. Resolved.
- Q: Should spawned sessions be denied the sleep commands? A (auto-pick): yes, `systemctl` with five sleep verbs, steering to `lyx hibernate request`. Resolved.
- Q: Add the `p` and `r` shortcuts? A (auto-pick): yes, with those names. Resolved.
- Q: Sandbox coverage? A (auto-pick): excluded, since the daemon hibernates the host. Resolved.
