# Support log — commit-signature-credit

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- Turning the block off by emptying its texts: implicit, and the board entry has the LoomYard line always present; a `credit` bool states it.
- Writing the ` - ` prefix into each config text: every text would repeat the list syntax; Go renders it once.
- Migrating `engine_trailers` values into `credit_engines`: they hold `Co-Authored-By:` lines, the text the task removes.
- Dropping the engine-name check so the template can carry a Codex line now: removes a typo guard for a line no run can produce yet.
- Keeping PR-Review's body as the bare description and signing only the merge commit: the board entry asks for one signature on both.

## Review rounds

_No rounds yet._

## Question ledger

- How is the block made optional? Self-picked: a `credit` bool, default true (D1).
- Does the template carry the Codex line? Self-picked: not until the Codex engine exists (D6).
- What happens to the old keys? Self-picked: retired and pruned, not migrated (D5).
