---
round: 1
focus:
  - "D9 in decision-record.md: it removes the transcript-shell expiry and counts a turn end whenever every output file is present. Settle whether this contradicts the recovery-strand turn-end rule from 7462c1dc5 (a turn end counts only when it is the strand's last and nothing keeps it waiting), and whether the stated bound covers what a stale shell can now hold open or let a turn end skip."
  - "D8 in decision-record.md: the read-only classifier downgrades a failed-batch fabric reference to a warning. Settle whether its bound covers shell constructs a segment split can miss (command substitution, backticks, here-documents, `env`/`xargs` prefixes, `git` options other than `-C`), and whether `git branch`, `config --get` and `cat-file` stay read-only under every listed flag."
  - "D1 and D2 together: D1 auto-rebaselines at the Webster row and accepts an in-flight batch's card edit as an amendment; D2 adds a plan-gate check against editing a done card. Settle whether the two agree on which edits are accepted, refused or stuck, and whether the acceptance criteria cover the in-flight amendment path that the Open risks name."
  - "D7 in decision-record.md: the refusal is keyed on `LYX_STRAND_NAME` with a deny-by-default role allowlist, and a session with no strand name runs. Settle whether the verb list is closed against verbs added later, whether a role outside the allowlist that legitimately runs fabric verbs exists, and whether the bound states what a missing or malformed strand name lets through."
  - "Goal versus Scope in decision-record.md: the Goal gives a count of defects and a list of named items, and Scope adds #476 as a further item. Settle whether the Goal agrees with the Scope and the Decisions on what is in the sweep, and whether every Scope bullet has a Decision and an acceptance criterion."
  - "D11 in decision-record.md: it carries a candidate-cause list for a fix it declines to predict. Settle whether that list is a requirement the plan writer needs or deliberation that belongs in support-log.md, applying the completeness-before-leanness test before any relocation finding."
  - "D12 in decision-record.md: the retry bound is two recoveries counted on `BatchState`. Settle whether the count survives a webster rebaseline or reset, and whether the `dead` classification and the committed-HEAD test are stated precisely enough to separate a retry from a refusal."
---

Seed focus only; each entry names a place to look and a question to settle, not a verdict.
The `Do not flag` list was applied: no entry asks for a rejected alternative in `decision-record.md`, the optional plan-writer notes or call-site enumeration.
The stencil's ban on gathering exact signatures and `file:line` citations means no entry asks for them.
