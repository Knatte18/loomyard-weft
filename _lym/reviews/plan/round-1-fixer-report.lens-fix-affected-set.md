```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran the searches behind each set finding to check the fix covers every member, not only the ones the review named.

Unchecked findings: none. F1 and F2 were both re-searched. F3 is not a set finding.

**F1 (claim that ensuring the runs strand replaces a loom status strand):** fully covered.
- The search finds `docs/overview.md:424` and `:687`, `internal/orchcli/doc.go:13`, `internal/orchcli/runs.go:19` and `:50`, and `internal/loomengine/driver.go:27`.
- After the fix, card 3 carries every one of them:
  - both overview.md lines through its docs bullet, now extended to the orch entry;
  - doc.go and `RunsStrandName` through their own Intent lines;
  - `runs.go:50` through `ensureRunsStrand` dropping its two replacement sentences;
  - `driver.go:27` through `LoomStatusStrandName`.

**F2 (comments in reed, orchcli and loomcli that say a session has one overview):** two members are still uncovered. The fix added the four the review named.

- **LOW** — card 2, Intent bullet "The doc comments of the rule's callers say the same in place of "the overview"". `Engine.AttachArgv`'s doc comment is left saying the single-overview fallback.
  - Evidence: `internal/reedengine/attach.go:43` reads "the newest live agent window, else the overview, never the shell".
  - That is the same fallback D2 changes to "else the lowest-index overview window". Card 2's Intent says the rule carries to attach.
  - Neither card 2's Edit list nor its caller-comment list names `Engine.AttachArgv`.
  - Fix: add `internal/reedengine#Engine.AttachArgv` to card 2's Edit list and to the caller-comment list.

- **LOW** — card 1's Intent, the sentence covering the Alt+Up/Alt+Down text. `bindingArgvs`'s comment is left calling the first window "the overview".
  - Evidence: `internal/reedengine/bindings.go:110` reads "forward onto the session's first window, the overview, and back onto the strands' window, the shell".
  - Card 1 rewrites the same sentence in `doc.go` to "the lowest-index overview", but no card edits `bindingArgvs`.
  - Fix: add `internal/reedengine#bindingArgvs` to card 1's Edit list, with an Intent line saying its comment calls the step's forward target the session's first window, the lowest-index overview.

Every other singular-overview comment the search found is already a card target:
- `openOwnWindowLocked` at spawn.go:258, `ReplaceStrand` at replace.go:41, and `render.AnchorOverview` and `overviewWindowInsertArgs` are in card 1.
- `reconcileLocked`, `killStrandPanes`, `FollowRun`, `followTargetWindow`, `relevantWindow` and `agentWindowPanes` are in card 2.
- The loomcli statements in bootstrap.go:27 and doc.go:62 belong to card 4, which I did not read.

Tool calls:
1. grep for the replaced-status claim across docs and internal — the F1 search; my context held only lines 424 and 687.
2. grep for singular-overview comments in reedengine, reedcli, orchcli and loomcli — the F2 search; my context held only the four lines the review named.
3. awk and grep mapping the remaining hits to their functions and to card coverage — my context lacked their enclosing symbols and card 4's text.
