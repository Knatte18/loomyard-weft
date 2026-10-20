# Support log — arbiter verdict quoting

## Interview

A darn run holds no interview; the writer read the board entry and the arbiter code and made the calls below itself.

## Rejected alternatives

- Tolerant `verbs` parsing that reads a mapping item back as `key: value` text: it covers only `verbs`, while a `: ` in a plain `reason` or `question` breaks the file the same way, and a ` #` silently truncates a plain scalar no parser can recover.
- A new `lyx loom` verb the arbiter runs to check its own verdict: it relies on the arbiter remembering to run it and adds CLI surface, while a shuttle gate runs regardless.
- Gating the tackler in the same change: no failure of its verdict is reported, and the seam change leaves it a one-line follow-up.

## Review rounds

_No rounds yet._

## Question ledger

- Parser tolerance, a self-check verb or a shuttle gate? Self-picked: a shuttle gate, plus the stencil's block-scalar rule (D1, D2).
- Gate the tackler too? Self-picked: no (Scope).
- Re-prompt budget? Self-picked: 2 (Auto-mode assumptions).
