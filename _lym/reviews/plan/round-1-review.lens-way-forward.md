```yaml
lens: way-forward
outcome: reported
```

Task: the way-forward lens. For each refusal, stop or recovery step the cards add, check the next step it names and whether that step can succeed.

**Per refusal: the next step, and whether it progresses**

- Card 1 `decision-heading-malformed`: rewrite as `### D<n> — <title>` or demote. Progresses.
- Card 1 `decision-id-duplicate`: renumber the decision and its mentions. Progresses.
- Card 1 `decision-not-in-acceptance`: name the id in a criterion, or add one. Progresses. A record with no `## Acceptance criteria` gets the `discussion-section-missing` way forward instead.
- Card 4 `citation-path-missing`: correct the path, or mark it ` (new)`/` (removed)`. Progresses; a marked token is not tested.
- Card 4 `citation-symbol-missing`: correct the symbol from `lym quarry glyphs <dir>`, or mark it. Progresses; I found no agent setting that denies `lym quarry`.
- Card 4 `citation-symbol-ambiguous` (one base name, several package directories): spell the symbol as a glyph. Progresses; the glyph resolves by its directory.
- Card 4 `citation-symbol-ambiguous` (quarry answer not split by build constraint): cite the declaring file by path. Progresses; D3 then tests the path.
- Card 4 quarry outage or git read error: returned error, so the gate halts as an infrastructure fault. This is an exempt transient error, not a writer refusal.
- Card 4 resolve-time `requireAbsRoot`/`requireSeam` for `WorktreeRoot` and `PlanIndex`: fail-loud wiring guards, which need no way forward.
- Card 5 `GuardMisplaced` for a seam line: move the line into the comment block directly above `func Test…`. Progresses for the test author.
- Card 5 `SeamMalformed`: rewrite as `//lym:seam <unit>#<Type>`. Progresses.
- Card 6 unmarked reflective test: add a `//lym:seam` line per reflected struct. Does not progress for a struct outside the module (finding 2).
- Card 6 `NilSeams` test missing nested glyphs: add the named glyphs. Progresses.
- Card 6 marker naming no struct: correct the glyph. Does not progress for a struct outside the module (finding 2).
- Card 7 `test-tier-unstated`: add an Intent line naming the file and its `//go:build` line. Progresses only when no tier is stated; it fails for two tags without `||` and for `untagged` beside a tag (finding 1).
- Card 8 `seam-test-unlisted`: Edit by member glyph, Uses, leave to the later editor, else Edit. Progresses.
  - A member glyph under Edit is not refused by `edit-test-file`.
  - `uses-later-target` is handled by the "else list it under Edit" clause.
  - The package self glyph route is accepted.
- Card 8 seam reader error (`*GuardScanError`) at a plan gate: returned error, with "a session that may edit tests" as the next step. That step names no one who can take it (finding 3).
- Card 9 `doc-go-unlisted`: Edit, Extends, Uses, leave to the later editor, else Edit. Progresses.
  - A `doc.go` declares no members, so `redundant-file-target` cannot fire.
  - `delete-before-reference` covers only later `Edit:` code (spec line 495), so it never refuses a Uses listing.

**Findings**

1. **MEDIUM** · design · `_lym/plan/07-test-tier-check.md` lines 42–43 (Intent: pass rule and finding text).
   The single way forward "add an Intent line naming the file and its `//go:build` line" cannot clear two failing states the card itself pins: two tags on a naming line without `||`, and `untagged` beside a tag.
   Card 7's test rows make `tier0 tier1` and `untagged` with `tier0` fail. All naming lines are counted together, so a new line adds a tag beside the conflicting one and the check still fails.
   The re-prompted writer is told to add, not to remove or join, and can spend the gate's re-prompt budget on a fix that cannot pass.
   - Evidence: `07-test-tier-check.md` lines 42–43 and the test rows at line 50; `decision-record.md` line 121 (the pass rule) and line 122 (the single way forward).
   - Fix: name the tags found on the file's naming lines, and give the conflicting cases their own clause: "join them with `||` when a helper several tiers share, else keep one", and "drop `untagged` or the tag".
2. **LOW** · design · `_lym/plan/06-seam-marker-scan.md` lines 36 and 41–43.
   A test that calls `NumField` on a struct declared outside the module (a `yaml.Node` or `time.Time`) has no admissible state.
   Unmarked, it is refused with "add a `//lym:seam <unit>#<Type>` line". Marked, it is refused because the glyph's unit directory declares no such struct, with "correct the marker's glyph". No worktree glyph names that struct, so neither step can succeed.
   - Evidence: card 6 line 36 (`envSeamStructs` lists no struct from outside the module because it has no worktree glyph); lines 41 and 43 (the two refusals); `decision-record.md` lines 94–96.
   - Fix: state the case as a bound or give it its own way forward, such as a marker for the nearest worktree struct, or reflecting through a non-test helper, which the scan does not see.
3. **LOW** · consistency · `_lym/plan/08-seam-test-check.md` lines 45–46.
   A seam-marker scan error at a plan gate is a returned error, so the gate halts instead of re-prompting. Its next step is "card 5's way forward, which a session that may edit tests takes".
   - Neither the Plan-Write session nor the Plan-Burler-Fix fixer may edit a test file.
   - The orch works only in its own worktree, per `CLAUDE.md`'s Worktrees rule.
   - So the halt names a step with no actor that can take it from the halted state, and the card names no command or role for it.
   - Evidence: card 8 lines 45–46; card 5 line 33 (the move-the-line way forward).
   - Fix: name who takes the step and how, such as the operator editing the marked test in the task worktree and re-running `lym loom validate-plan`, or state the halt as the parent's.

**Tool calls**

- Grepped the plan spec for the `delete-before-reference` definition and agent settings for a `lym quarry` deny: I needed to know whether a Uses listing after an earlier Delete is refused, and whether a discussion agent may run `lym quarry`. Neither was in my context.
- Grepped `contracts` and `internal` for agent permission or deny settings naming `lym quarry`: the first call found no settings file to check.
