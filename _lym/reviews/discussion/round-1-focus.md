---
round: 1
focus:
  - "D5, attack surface: the status line verb takes an account name argument and writes `<Bolt>/.lym/claude/usage/<account>.json`; check whether a Decision states how the name is validated against D1's `[a-z0-9-]+` and what an unlisted or malformed name can write or skip."
  - "D5, attack surface: the injected `statusLine` command replaces the operator's own in agent panes; check that what it can skip or override, and what bounds it, is stated in a Decision and not only in Auto-mode assumptions."
  - "D2 and D3, attack surface: the pick's escape routes (a record kept after its account leaves the list, the all-short fallback, the macOS skip of the credentials stat, a recorded empty account name); check each states what it lets through and what bounds it."
  - "Deferred facts: D3's macOS Keychain claim and D1's open risk on the Ultra five-hour window; check whether any Decision, Constraint or criterion leaves a fact the design rests on for Plan or an implementer to find out, or hands downstream a halt on discovering it."
  - "Consistency across D2, D3, D5 and D6: a record with an empty account name against D5's listed-account condition, D3 step 4's 'blocking windows all reset soonest' against a candidate with several blocking windows, and D6's known-directory set against D2's resume search set."
  - "Acceptance criteria: each is behavior a test can check or a `Review:` bullet; check D4's unset-in-default case, D7 and D8 for criteria that no test could observe or that duplicate another bullet."
  - "Relocation: apply the completeness-before-leanness test to D9's operator wiring steps, D1 to D8 Rationale paragraphs and the Open risks lines, and flag only deliberation narrative that carries no requirement the plan writer needs."
---

Seed focus only; the reviewer still judges the whole pair against the rubric.
No entry above touches a `Do not flag` item: none asks for a Notes subsection, rejected alternatives in the record or a call-site enumeration.
