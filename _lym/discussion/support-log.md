# Support log — reed-status-buttons

## Interview

Autonomous run: no operator answers; every pick below is an auto-pick recorded in the question ledger.
No advisors were seated.

### Turn 1 — exploration

Read the board entry, the board list, note `board-agent-button` and `lym loom peers`; no `Decision for reed-status-buttons:` note exists.
Peer `machine-switch` rules out a reed button of its own and touches no reed or desktop code; no conflict.
Read reed's bar, bindings, follow, check mark, config and geometry pins, `hubgeom.ReedGeometry`, `desktophook`'s config, `configengine`'s layer rules and the help of every command the default set runs.

Facts settled:

- tmux status formats are one global array; reed pins them with `-g` (`internal/reedengine/windowsize.go`).
- `range=user|X` caps `X` at 15 bytes (tmux 3.6 manual).
- The machine layer applies only to hub-wide modules; reed is per-worktree; `desktop.yaml` is hub-wide with machine keys.
- `lym batten cancel` runs only from the prime or a beam (`ResolveBattenPrime`); `lym tender pause`, `lym ide spawn`, `lym orch distill|refresh` and `lym shuttle open` run from any worktree.
- `lym tender` reads the caller from `LYM_STRAND_NAME`; reed's server env scrub strips engine keys only, so a server started from an agent pane can carry a strand name into `run-shell`.
- `lym ide spawn` accepts the prime's name.

### Turn 2 — design picks

Questions formed and settled by auto-pick (ledger Q1 to Q10): where machine buttons live, what `{beam}` means, the new-agent role, how a click reaches a per-session command, confirm flow, rendering timing, label safety, caller identity, `follow`'s place and the beam set.

### Turn 3 — lens review

All six forks returned a report; findings judged together, duplicates merged.

- walkthrough 1, existing-paths 1, bound-breaker 1 (`{strand}` from `@strand` is the role only): fixed — new pane option `@lym_strand_name` with the full name (D6).
- existing-paths 2 (display-only invariant of pane options and `@lym_follow` doc lines): fixed — D5 names the decision-bearing options and the doc lines that change.
- bound-breaker 2, interactions 3, walkthrough 4 (confirm path carries `run` text through tmux parsing; `{strand}` mismatch): fixed — hash of the spec `run` plus a grammar-checked strand name cross `confirm-before` (D7).
- interactions 2, walkthrough 2, bound-breaker 4 (verb needs the shell but reads no config): fixed — shell path in the spec (D5).
- walkthrough 3, interactions 4, bound-breaker 7 (log location): fixed — `run-shell -c '#{session_path}'` (D7).
- walkthrough 5 (`run-shell` output opens view mode): fixed — the pinned line discards the verb's output (D7).
- interactions 1, bound-breaker 5 (attach skips validation; invalid button fails every spawn): fixed — validation at render, invalid entries skipped with a Warn for both sources (D2).
- interactions 5 (`#` in filled `confirm` values): fixed — `#` doubled in confirm fills (D6).
- interactions 6, existing-paths 3, bound-breaker 6 (older binaries re-pin status formats): fixed — status pins join the stale-build re-pin, remaining window bounded (D1).
- bound-breaker 3 (spec in a socket-writable option): fixed — stated in D7's bound with the tracked-config case; Open risks line dropped.
- bound-breaker 8, walkthrough 6 and 7 (`ParentOf` cost and prime location): fixed — `{beam}` resolved only when used, over the prime's location, slug mapped to path, cost accepted (D4).
- bound-breaker 9 (machine shape check unverified): fixed — checked: each group is one list unit and the shape check is mapping-or-not (D3); edit route named.
- bound-breaker 10 (CANCEL removes the clicking session or worktree): fixed — failure message to a gone client only logged (D7).
- bound-breaker 11 (spec and rows written separately): fixed — one chained tmux invocation (D5).
- existing-paths 4 (rows not refreshed at each op; beam marked after boot): rejected — assumption kept; attach is the refresh point, now stated in D5.
- existing-paths 5 (Windows template): fixed — D8 covers both templates.
- board-scope LOW (template versus hub default): fixed — one line in D2.
- leanness 1 (assumptions restate decisions): fixed — pointers only.
- leanness 2 (D3 rationale deliberation): fixed — trimmed to the category claim.
- leanness 3 and 4 (empty or duplicated constraints): fixed — cut or folded to one pointer.
- leanness 5 (Open risks line is a writer's call): fixed — folded into D7's bound.
- leanness 6 (vague D8 criterion): fixed — CANCEL check named.
- leanness 7 (tier hint): fixed — cut.
- leanness 8 (keep `status.c` evidence): no change needed.

## Rejected alternatives

- Machine additions through reed's local layer: per-worktree, and it refuses executable keys.
- Making reed's `buttons` key hub-wide or a new hub-wide `buttons.yaml` module: splits reed's config from the operator's "one entry in reed's config", for no gain over `desktop.yaml`'s existing machine layer.
- Buttons as a per-name map so layers add by name: loses display order and departs from the operator's list syntax.
- A per-session `status-format[2]`: a session-scoped index hides the global lines 1 and 2.
- One `if-shell` branch per button index in the global binding: bounded count, rebinding on every config change.
- The click verb re-reading config at click time: the clicked index can drift from the drawn row after an edit.
- `confirm-before` running the command directly: loses output logging and failure display.
- Escaping `#` in labels: two expansion passes make the escape fragile; refusing `#` is simpler.
- Agent-window test by window markers: the active pane's `@strand` is exactly what `{strand}` needs.
- `{beam}` as the beam's slug: the one default that needs it needs a directory.
- Failing `lym reed up` on an invalid button: one bad entry would stop every agent spawn of the hub.
- Carrying the filled `run` text across `confirm-before`: tmux parses and expands it, so `#`, braces and quotes break.
- Holding the spec in reed state under `.lym`: an agent can write its worktree as easily as the socket, so it closes nothing.
- Asking the role for the new-agent button: needs a prompt feature nobody asked for.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): where do machine-added buttons live? `desktop.yaml` `buttons`, `CategoryMachine` — the only hub-wide machine layer for desktop facts.
- Q2 (auto-pick): `{beam}` value? The beam's worktree path; the prime's for a top-level task.
- Q3 (auto-pick, settles the entry's Open): new-agent role? Fixed `agent`, `opus[high]`.
- Q4 (auto-pick): click to command? A `lym reed button` verb reading a session spec option.
- Q5 (auto-pick): confirm flow? `confirm-before` re-invoking the verb with `--confirmed`, checked against the spec.
- Q6 (auto-pick): when is the row rendered? At the existing status pins, boot and attach pre-flight.
- Q7 (auto-pick): label safety? Refuse `#` and control characters in `label` and `confirm`.
- Q8 (auto-pick): caller identity of button commands? Strand-name and parent variables removed, so the click reads as the operator.
- Q9 (auto-pick): `follow`'s place on line 3? First.
- Q10 (auto-pick): beam set content? `code`, `term`, `agent`, `distill`, `refresh`.
- Q11 (auto-pick, after lens review): invalid button handling? Skip with a Warn at render for both sources, never fail a reed verb.
- Q12 (auto-pick, after lens review): source of the full strand name? A new pane option `@lym_strand_name`.
