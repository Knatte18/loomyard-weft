```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch on one concrete input. I found one defect and one nit.

**Per function or branch (case, result):**
- `cardIDTokens`:
  - `x5-foo` has a left neighbour that is an id byte, so it is skipped. Correct.
  - `5-foo-bar` has a right neighbour that is an id byte, so it is skipped. Correct.
  - `5-foo-` followed by a space or end of line is accepted. Correct.
  - `05-foo.md` gets width 2. Correct.
  - `5-foo.md` also gets width 2, so a file-form name stays padded. Correct.
  - `005-foo` keeps width 3. Correct.
  - `3-5-foo` is skipped at the second number because its left neighbour is `-`. Correct.
- `rewriteCardIDs`:
  - Several tokens on one line, with the offsets taken from the original line, are replaced correctly.
  - With width 0, `%0*d` prints a plain number. Correct.
  - A swap such as 5→6 and 6→5 is computed from the pre-renumber map in one pass and is not applied twice. Correct.
- `applyRenumber`, moved cards: the rewrite runs after `moveCardBytes`. Because `moves` is keyed by old number and slug, an already-renumbered Card Index id such as `6-foo` cannot match unless two cards share a slug. Correct, given unique slugs.
- `applyRenumber`, stayed cards: cards that keep their number are skipped when they move or are the removed card. A file is journaled only if a rewrite changed it. The journal is written before any step runs, so a failed step is restored from the journal. Correct.
- `applyRenumber`, rewrite order: the refs are sorted by file and then line before they are returned. Correct.
- `proseMentions`:
  - Its caller at `renumber.go:352` passes `planAfterRenumber`, so the scan sees post-renumber ids and slugs.
  - `maskCardIDs` blanks both rewritten and unmoved ids, so `Card 5-foo` is not read as a bare number. Correct.
- `ParseReport` / `validateReportFields`:
  - An empty side is refused. Correct.
  - `old == new` is refused. Correct.
  - A missing `renames` key leaves no entries. Correct.
  - The `deviations` key is still accepted. Correct.
  - A missing `reason` is also refused. See F1.
- `distill` and the digest:
  - `Renames` carries through on all three statuses, and `omitempty` makes an older state read as none. Correct.
  - Recover-batch builds its digest from `ParseReport` at `recoverbatch.go:793`, so the renames reach `PersistRecoveryTerminal` at `recoverbatch.go:698`. Correct.
- `DetectDrift`, the `queue` closure:
  - A declared Rename pair is skipped. Correct.
  - An old glyph no pending card references is logged only. Correct.
  - A pair the exact tier already renamed is skipped through `exactOld`. Correct.
  - A repeated `Old` among the reported pairs keeps the first. Correct.
- `DetectDrift`, the sweeps: a queued old glyph is excluded from both the deleted-symbol sweep and the evidence-tier sweep. A missing glyph with no reported replacement still reaches the deleted-symbol finding. Correct.
- `DetectDrift`, the new glyph does not resolve: the rewrite stays on disk, a blocking finding is produced, and the amendment is still appended with tier `reported`. This matches the exact tier and the acceptance criterion (`drift.go` around line 270).
- `postBatchChecks` with nil or empty renames: this gives an empty slice and no repair. Correct.

**Findings**

**F1 — LOW — design — `internal/websterengine/report.go:101`**
`validateReportFields` refuses a `renames` entry with an empty `reason`. The decision record and acceptance criteria require refusing only an empty side or `old == new`. A fork that reports a correct rename but forgets the reason then fails the whole report. That sends the batch to the recover-batch path instead of applying the rename mechanically. `Reason` is never read after parsing, apart from being copied into the digest.
Evidence: `report.go:101` against decision-record "ParseReport refuses an entry with an empty side or with `old` equal to `new`".

**F2 — NIT — design — `internal/websterengine/report.go:101-107`**
The emptiness check trims whitespace, but the untrimmed `Old` and `New` are what get passed on. The `Old == New` comparison is also untrimmed, so `a` and `a ` count as different. Quarry glyph ids never carry whitespace, so this is only an edge. YAML plain scalars are already trimmed.

**Tool calls**
1. Read `internal/planglyph/drift.go` lines 195-300 to see what happens when a reported new glyph fails to resolve (the diff shows only the changed hunks).
2. Grepped `internal/planparser/renumber.go` to find where `proseMentions` is called and with which plan (line 352, `planAfterRenumber`).
3. Grepped `recoverbatch.go` and `classify.go` to confirm recover-batch builds its digest through `distill`/`ParseReport`.
