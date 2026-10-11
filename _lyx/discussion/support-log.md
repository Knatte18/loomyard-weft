## Interview

No operator was available, so the interview was a self-run pass: read the board entry and its issues (#505, #509, #517, #522, #511, #500, #507), read the base code at 800a0e892, and settle each point with a recommendation.

- **Gate shell (item 1).**
  Base already never expires a shell (`background_shell_wait_min` only logs), but a gated run with every output file present and only unawaited shells outstanding finishes Done at once and strand removal kills the shell.
  Only `lyx webster recover-batch` is an awaited prefix.
  Picked: add `lyx gate test` to Master's awaited prefixes; keep lyx from killing shells; strengthen the ended-shell record.
- **Master self-fix (item 2).**
  Master's stencil already says "Do NOT localize or fix the failure yourself", but the message Master acts on is shuttle's generic "read it, fix every finding, and end your turn".
  Picked: a per-entry re-prompt note on `GateEntry`, set by the verify entry.
- **Second recovery (item 3).**
  `uncommittedPathsBlock` groups by `loadRunWrites`, which audits only Master sessions (`MasterSessionID` and each batch's `SessionID`) and their forks.
  A recovery strand runs in its own session, which no record keeps, so its writes read as foreign.
  Picked: record recovery session ids on the batch record and include them.
  `recover-batch` returns `output.Ok` (exit 0) for a terminal `stuck` or `dead` digest; picked a non-zero exit with the digest fields kept.
- **Recovery gates (item 4).**
  The shared implementer body has a sandbox rule banning mutating `lyx fabric` verbs and a raw-go note; nothing says the card and batch gates are allowed or what to do with tmux steps.
  Picked: a section in the recovery prefix only.
- **Create check (item 5).**
  `donecheck.go` reports "Create target %q still does not resolve" with no status.
  quarry v0.2.1 resolves unexported identifiers and test-file members, so the resolver is not the limit for those shapes.
  Picked: richer detail plus pins.
- **Build-tagged members (item 6).**
  quarry v0.2.1 has no build-constraint handling.
  `DoneChecks` accepts only `found` and `multipart` for Create; `statusFindings` blocks any `ambiguous` at Plan time.
  Picked: one classifier in `planglyph`, used by both.

## Rejected alternatives

- **A Master verb that spawns the verify fixer.**
  A fixer has to be an in-session fork that inherits Master's context, and only the Agent tool spawns one; a verb would start a cold strand with a different cost and context.
- **Stopping the gate shell from lyx and recording the discard.**
  lyx kills no shell (the shell belongs to the provider's process tree, and `PATTERN-shuttle-provider-seam` keeps provider specifics out of shuttle); holding the turn end avoids the discard instead.
- **Substring or word match for awaited gate shells.**
  A prefix rule is the existing mechanism; widening it for every awaited prefix changes `recover-batch` matching too.
  The stencil asks for a plain command instead.
- **Starting a second recovery from a clean tree with the first attempt's diff offered.**
  It throws away work the strand could check and keep, needs a stash or reset path, and the destructive step would have to live in a guarded verb (`PATTERN-no-denied-recovery`).
- **Honoring build constraints in quarry.**
  quarry is an external module at v0.2.1; a change there needs a release and a version bump, and the plan gate would still need the same decision for older answers.
- **Accepting any `ambiguous` Create answer in DoneChecks.**
  It would pass a real duplicate declaration; disjoint constraints are the only evidence of a legal split.
- **Putting the recovery allowed-commands section in the shared implementer body.**
  A fork inherits Master's context and the issue is about the cold start; one stencil edit in the prefix is enough.
- **A rule only in Master's stencil for item 2, or a pointer inside the findings file.**
  The stencil rule already exists and was not enough; the findings file is read after the message that says "fix".

## Review rounds

_No rounds yet._

## Question ledger

1. Does a running gate shell hold Master's turn end, or does lyx stop it? Resolved, auto-pick: hold, by awaited prefix `lyx gate test`.
2. Should an ended shell's record say its result was not read? Resolved, auto-pick: yes, one clause in the shared warning wording.
3. Where does the "do not fix" rule go for Master? Resolved, auto-pick: `GateEntry.RepromptNote`, set by the verify entry.
4. Attribute a second recovery's files by batch, or start clean? Resolved, auto-pick: by batch, through recorded recovery session ids.
5. Which recover-batch outcomes exit non-zero? Resolved, auto-pick: terminal `stuck` and `dead`; the cause of #522's exit 0 was not reproduced.
6. Where do the always-allowed gates go? Resolved, auto-pick: the recovery prefix only.
7. Is the Create check's resolver at fault for #511? Resolved, auto-pick: not for unexported or test-file members on quarry v0.2.1; the diagnostic detail is the fix. Open: the real root cause of the two refusals.
8. How is "partitioned" decided? Resolved, auto-pick: pairwise-disjoint constraints over `//go:build` plus filename suffixes, via `go/build/constraint`, failing closed.
9. Does a partitioned glyph pass at Plan time too? Resolved, auto-pick: yes, in `statusFindings`; a Create of an existing partitioned name still blocks.
10. Create-after-delete in one card (#507 second half). Resolved, auto-pick: out of scope, listed under Open risks.
11. Revive a dead batch through `goto` (#511 third ask). Resolved, auto-pick: out of scope.
12. Base exposure. While probing resolver behavior, `lyx quarry resolve` ran against the checked-out HEAD tree, which is ahead of 800a0e892, and echoed the doc comments of two `planglyph` functions as 1e79825ff left them.
    Every decision above rests on base file reads (`git show 800a0e892:<path>`) and the quarry v0.2.1 module source; the exposure is recorded here so the reviewer can weigh it.
