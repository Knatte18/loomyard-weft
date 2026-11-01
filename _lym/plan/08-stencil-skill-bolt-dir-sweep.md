# Card 8 — stencil-skill-bolt-dir-sweep

**Edit:**
- `//contracts/stencils/burler/burler-review-format.md`
- `//contracts/stencils/burler/burler-step-1-explore.md`
- `//contracts/stencils/burler/burler-step-fix.md`
- `//contracts/stencils/burler/burler-step-review.md`
- `//contracts/stencils/burler/burler-template-fixer.md`
- `//contracts/stencils/burler/burler-template-reviewer.md`
- `//contracts/stencils/darn/darn-template-write.md`
- `//contracts/stencils/drift/drift-template-baseline.md`
- `//contracts/stencils/drift/drift-template-verify.md`
- `//contracts/stencils/friction/friction-template-reflection.md`
- `//contracts/stencils/landing/landing-template-conflict.md`
- `//contracts/stencils/landing/landing-template-describe.md`
- `//contracts/stencils/landing/landing-template-fixer.md`
- `//contracts/stencils/landing/landing-template-fix-review.md`
- `//contracts/stencils/landing/landing-template-pr-review.md`
- `//contracts/stencils/loom/loom-template-plan.md`
- `//contracts/stencils/orch/orch-template-role.md`
- `//contracts/stencils/sampler/sampler-template-agent.md`
- `//plugins/ly/skills/board/SKILL.md`
- `//plugins/ly/skills/INDEX.md`

**Create:**
- `plan:internal/lymcwd#TestEnforcement_BoltDirNameDeclaredOnce` -> `func TestEnforcement_BoltDirNameDeclaredOnce(t *testing.T)`

**Uses:**
- `plan:internal/lymcwd#BoltDirName`
- `internal/testkit/scankit#Root`

**Intent:** Finish D9: no stencil or skill spells the Bolt directory, and a guard keeps the literal to its one declaration.

- Each stencil's leading comment "seeded to `<hub>/_board/_lym/stencils/<group>/`" becomes "seeded to the Bolt checkout's `_lym/stencils/<group>/`".
  `orch-template-role.md`'s two mentions of the hub-wide rules file name it "the Bolt checkout's `_lym/orch/rules.md`", the path `lym orch rules` writes, so the draft step still tells the orch which file to copy.
  The `board` skill's description and its "never touch" rule, and `INDEX.md`'s row repeating that description, say "the board's JSON files in the Bolt checkout" instead of `_board/*.json`.
  No marker, Go-filled value or Go-chosen fragment changes, so no stencil test changes.
- `TestEnforcement_BoltDirNameDeclaredOnce`, in `internal/lymcwd/enforcement_test.go` (build tag `tier0`, the file's own), carries `//lym:guard` on the line directly above its func line, as `TestEnforcement_GeometryLiterals` does in the same file, since it reads files outside its own package (PATTERN-test-speed); it walks the repository from `scankit.Root(t)` and fails naming each file and line where the value of `BoltDirName` appears as a whole token (no letter, digit or underscore on either side, so `_boardroom` passes) in:
  every `.go` file, test files included, skipping directories named `.git`, `testdata`, `vendor`, `_lym` and `.lym`;
  and every `.md` file under `contracts/stencils`, `contracts/specs` and `plugins`.
  It exempts `internal/lymcwd/anchor.go`, where it asserts exactly one occurrence, the declaration's value, and `internal/lymcwd/enforcement_test.go`, whose geometry test spells the token it polices.
  The test builds its pattern from `BoltDirName` with `regexp.QuoteMeta`, so it spells no literal itself, and it first checks its matcher against planted strings (`"x/_board/y"` built from the constant matches, the constant plus `"room"` does not) so a vacuous pattern cannot pass.
  It passes only after cards 2 through 6 have moved every Go file to the constant, which is why it lands here.

**ImpactSummary:** Prose edits to 18 stencil comments and two skill files, read by no Go test; one new guard test in `internal/lymcwd`.
