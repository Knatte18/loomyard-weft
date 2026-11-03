---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/planparser/validate.go:1377
    origin: lens:code-correctness
    summary: >-
      the remedy fires for a //-escaped path that canonicalization turned into a self glyph, and its wording blames a glyph spelling the writer never wrote
    fix: >-
      In internal/planparser/validate.go, reword the Sprintf in rootRelativeGlyphRemedy so it does not assert that the writer spelled a glyph, for example "; root: is never joined to a glyph or a //-escaped path, so this names the worktree-root file %q, while the file root: joins is %q; to name it, spell it as %q, or drop the // escape or the trailing # so root: joins the path %q". Add a row to the new subtest in internal/planparser/validate_test.go in which the card ref is //guide.md under root docs with docs/guide.md present, and assert the new wording.
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/specs/loom-plan-checks-spec.md:64
    origin: lens:code-affected-set
    summary: >-
      the spec says the remedy needs the root:-joined path to be present, but the code also accepts a Create target, a Rename destination and a part path
    fix: >-
      In contracts/specs/loom-plan-checks-spec.md line 64, change "whose `root:`-joined path is present" to "whose `root:`-joined path is present on disk or created by a card", matching the satisfied closure in checkPathMissing (internal/planparser/validate.go:1311).
  - id: F3
    severity: LOW
    class: design
    location: internal/planparser/validate_test.go:1517
    origin: lens:code-tests
    summary: >-
      no row covers the guard that gives a path-shaped, non-glyph ref under root no remedy
    fix: >-
      In internal/planparser/validate_test.go, add a row to the table of the subtest named self glyph spelled relative to root, names the worktree-relative remedy. It uses an Edit card whose ref is a path that stays path-shaped under root docs, such as a slashed extensionless directory, with the file that root joins it to present on disk, and asserts the path-missing detail holds no spell it worktree-relative remedy. It fails when the ParseRef guard in rootRelativeGlyphRemedy (internal/planparser/validate.go:1365) is removed.
---

### [LOW] Remedy wording misdescribes a //-escaped path

**Location:** internal/planparser/validate.go:1377
**Issue:** Canonicalization turns `//guide.md` under `root: docs` into `guide.md#` before `checkRef` runs (`normalize.go:42-44`, `169-193`). When `docs/guide.md` exists and the worktree-root file does not, `rootRelativeGlyphRemedy` cannot tell the two spellings apart and says a glyph is never joined to `root:`, to drop the trailing `#`. The writer wrote no glyph and no `#`. The verdict is right and the hint still names a valid file, so only the text is wrong.

### [LOW] Spec says present where the code says satisfied

**Location:** contracts/specs/loom-plan-checks-spec.md:64
**Issue:** The code builds the remedy when `satisfied(joined)` holds, which is true for a path on disk, a `Create` target, a `Rename` destination or a part path (`validate.go:1311-1313`, `1370`). The spec line and the landing summary say only that the joined path is present.

### [LOW] Non-glyph guard is unpinned

**Location:** internal/planparser/validate_test.go:1517
**Issue:** The only `root:` rows use the glyph ref `guide.md#`. The early return on a `ParseRef` error (`validate.go:1365`), which keeps a path-shaped ref from getting a glyph remedy, is behavior the diff adds and no row fails without it.

## Rejected

- Scope against the entry's Goal, a shared parser and a bouncer that rejects what Webster rejects: the plan gate (`loomshed.ValidatePlan` through `index.ValidateFormat`) and Webster's begin-batch (`index.ValidateDispatch`) both reach `planparser`'s checks and `planglyph`'s resolve pass, and the bouncer already refuses `root:`-relative `README.md#` with `path-missing`, so the diff makes no change the entry still asks for and licenses none beyond the message and two spec lines.
- Empty-anchor handling under no `root:` and under `language: none`: the remedy returns `""` for both, as `diskPathForRef` is not-ok under `none` and the root test returns early for an absent root or `.`, so the verdicts and details match what the spec states.
- Assertion pins only the first remedy clause (code-tests lens): a new assertion on the static prose of a message is not a coverage gap under the rubric's test-economy rule.
- No row for a joined path satisfied through a `Create` target or `Rename` destination (code-tests lens, NIT): the closure `satisfied` is shared with the verdict, whose union paths other rows cover, so a row would repeat that coverage.
- Behavior change in `checkPathMissing` from inlining `report`: `Check`, `Card` and `Ref` are built identically at the one remaining site, and no test, stencil or spec matches the old `Detail` sentence beyond a file-name substring.
- Declared PATTERN changes and `Pattern-Departure:` trailers: the directive declares none, the diff touches neither `PATTERN.md` nor `pattern/`, and the commit carries no trailer.
