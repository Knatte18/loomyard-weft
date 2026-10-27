# Support log — creel-hook-delivery

## Interview

A darn run holds no interview: the writer read the board entry, the research report at `docs/research/creel-hook-delivery-2026-10-25/report.md` and the code, and made every call itself.

## Rejected alternatives

- One hub-wide flag file listing the strands with mail: the hook would need `grep` whenever any strand of the hub holds mail, which the orch often does.
- Flag file named by the bare strand name: `:` is not a valid file-name character on Windows.
- A new environment variable carrying the flag path into the pane: reed would take on a creel concern, and the board directory the engine is told already locates it.
- The hook verb printing the existing JSON envelope on a refusal: Claude Code parses a `PostToolUse` hook's standard output as hook JSON.
- Claiming every unread message with no budget: a 64 KiB body, or many of them, would flood one tool result's context.
- Leaving the pane path's notice unreserved while it types: a hook read in that window would hand the same notice out twice.
- Building the hook command in each spawning module, as the orch does for its resume context: every Claude strand needs it, and there are many spawners.

## Review rounds

_No rounds yet._

## Question ledger

- Should the hook deliver under a hold? Self-pick: only the hold owner's mail, as the pane path does.
- Should the hook deliver under a pause? Self-pick: no, a pause holds every delivery.
- Which engine renders the output? Self-pick: an optional engine capability, chosen by the engine name the hook command passes.
- Where does the hook run `lym` from? Self-pick: `$CLAUDE_PROJECT_DIR`, so a session that changed directory out of the hub still resolves it.
