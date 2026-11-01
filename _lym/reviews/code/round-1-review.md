---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shellguard/rows.go:183
    summary: the new inline comment in refuseRawRow explains the notice by comparison to the raw go row, which the comment rules keep out of a single symbol's comment
    fix: reword the comment above Notice in refuseRawRow to stand alone, for example "The notice names only what is refused and the scopes it adds; the gate rule itself is stated by another row's notice, see the package doc." and keep the cross-row relationship only in the package doc paragraph in internal/shellguard/doc.go.
  - id: F2
    severity: NIT
    class: consistency
    origin: handler
    location: internal/shellguard/doc.go:18
    summary: the new paragraph says each row's steer carries the full rule, but only the raw go and refuse-raw steers do
    fix: in the doc.go paragraph replace "while each row's steer carries the full rule" with "while the steers of the raw go and refuse-raw rows each carry the full rule".
---

### [LOW] Inline comment explains a notice by comparison to another row

**Location:** internal/shellguard/rows.go:183
**Issue:** The comment above `Notice` reads "The raw go row, installed for every run, states the gate rule in the notice, so this one names only what it refuses and the scopes that row's notice leaves out."
It is correct against the code, but it is written in terms of another named row and "this one".
The repo's comment rules want a symbol's comment to be correct to a reader who knows no other symbol, and such relationships to live in the package doc, which this diff already extends.

### [NIT] Package doc overstates which steers carry the rule

**Location:** internal/shellguard/doc.go:18
**Issue:** "each row's steer carries the full rule" reads as every row.
The python, computed-rm, machine-sleep and publish steers carry none, and the answer key's wording ("each row's steer ... keeps the full rule") refers to the two rows that had it.

## Rejected

- Lens code-tests, LOW: no test counts "Gate rule" in the assembled system prompt. Rejected: `buildDenyNotice` is unchanged and joins the installed notices, which `TestBuildDenyNotice_MatchesInstalledDenies` pins; the row-level count in `TestCheckTexts` plus that join fully determine the prompt, so the extra assertion would duplicate coverage.
- Lens code-tests, NIT: the exact-string `wantNotice` in `TestCheckTexts` is brittle static prose. Rejected: the lens itself calls it advisory; the exact string is what pins "says only what it refuses", and it does not duplicate the count test.
- Focus questions settled with no defect: the raw go notice still carries the whole rule (`rows.go:161`, `210`, unchanged); the refuse-raw steer is byte-identical to the old one, built as `refused + ". Gate rule: ..."`; both rows are `SwitchNone`, so `buildDenyNotice` emits the refuse-raw notice only together with the raw go notice and the rule is stated once, never zero times; the refuse-raw notice names the refused prefixes, the gate verb and the tier and card scopes, which is enough to act on and within "says only what it refuses"; the summary's claim that the golden settings are unchanged holds, since `TestBuildSettings_MatchesGolden` renders with no refuse list; no PATTERN entry is declared changed and `PATTERN-shell-guard-table` is unaffected.
