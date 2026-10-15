# Webster's Master reads the plan and rules, never code

Webster's Master template told the Master to read the codebase once up front so every implementer fork would inherit that orientation.
Forks rewrite the code, so every fork after the first inherited stale reads.
The Master now reads only what every fork shares and what stays stable for the whole run, and each fork reads the code its own cards need, fresh, when the card starts.

## What changes

- The Master reads `00-overview.md` once, the `pattern/` background files its constraints section points at, and the gate findings file a `Gate findings recorded at …` message names.
  It reads no card file, no source, test or other doc outside those, and runs no grep or quarry over the code.
  `## What you never do` forbids any other read, with the reason that forks rewrite the code and a later fork would inherit a stale read.
  `CLAUDE.md` needs no instruction: Claude Code loads it into the Master's context.
- The fork prefix says the fork inherits the plan's framing and the repo's rules and no code, and the title no longer implies a code orientation.
  The "You are the IMPLEMENTER" section is unchanged.
- The three live FRESH-READ fragments (plain, pack, pack-card) drop the "one Master read before forking you" framing.
  They now say no code reaches the fork from Master, and only its own reads of code taken in this batch are current.
  The plain fragment's "re-read" becomes "read", so it holds for the cold recovery strand, which renders it too.
- The recovery prefix orients from `00-overview.md` and the rules, reads code per card, and no longer claims an in-session fork holds an orientation from Master.
- The verify-fix stencil says the inherited context holds no code, so the fixer reads the code behind each finding itself.
- `internal/websterengine/doc.go`, `docs/overview.md` and `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` no longer say the Master reads the codebase.
- The pins in `contracts/stencils/claims_test.go` and `internal/websterengine/template_test.go` assert the Master's read set, the forbidden reads and the absence of any inherit-code claim.

No Go behavior changes: no new hook, audit check or verb.
The Master is held to the read set by its prompt alone.

## Design decisions

- `webster-fragment-fresh-read-pack-inline.md` stays byte-identical, because its banner makes it the layout measurement baseline.
  Its "one Master read before forking you" sentence is a may-be-stale hedge, not a claim that Master read code.
- The rule lives in the Master template and the `websterengine` `doc.go`, not in `PATTERN.md`, because it binds one role and is not cross-cutting.
- The Master template has no conditionals, so its rules read names the constraints section and reads as vacuous when the pattern directive renders empty.

## To check by hand

- Rules read in the Master's opening paragraph: check that the paragraph does not name the rules or `pattern/` files unconditionally, since with no `PATTERN.md` there is no constraints section to condition them on.
- Verify tier scope: check that the verify tier skipping the `tools/` sandbox package does not leave the edited `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` unchecked.
- Orientation section of the Master template: check that `## Orientation` does not still license reading every state file rather than the named set.
- Re-measure after landing (not run).
  Run `tools/forkaudit` over the first Webster runs made on a binary deployed from this change's landing commit.
  Compare per-fork total reads and failed Edit calls (the stale-read proxy) against the 2026-10-12, 2026-10-13 and 2026-10-15 audits, not calls before the first edit.
  Check the Master's own reads in those runs as the compliance check for the new read set.
  Forks explore more by construction, so a rise in per-fork reads is expected and is not a regression.
  If this lands with no orch review, nothing schedules the check: file a board note of kind `note` and label `audit` naming `webster-master-no-code`, due at those first runs.
- A Webster run in flight at deploy mixes the old and new prompts, since stencils are read at call time.
  The fresh-read rule makes forks read their own files either way, so the mix costs wording accuracy, not correctness.
- A plan whose cards edit `PATTERN.md`, a `pattern/` file or `CLAUDE.md` leaves the Master's copy stale for later forks.
  The card that edits such a file names it as a target, so its fork reads it fresh, and later forks get the changed rule through the plan's `## Shared Decisions`.
