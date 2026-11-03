---
round: 1
focus:
  - "Scope against the board entry's Goal: it asks for one path-with-anchor parser shared by the plan bouncer and `lym webster` validation, and for the bouncer to reject what Webster rejects. Check whether the diff in `internal/planparser/validate.go` does that, or only appends a remedy to the `path-missing` detail. Also check whether the Webster side (`ParseRef`, the Webster path check) was touched or already shares the code."
  - "Empty-anchor handling: the entry's example is `<dir>/README.md#`, a bare `#` after a markdown file, with a repo-relative path and with a path relative to the module directory. Settle how the diff treats `X.md#` under no `root:`, under `root:` other than `.`, and under `language: none`, and whether the remedy text is right for each."
  - "`rootRelativeGlyphRemedy` in `internal/planparser/validate.go`: check the `glyph.Self` result and the `path.Join(root, mapped)` result against `mapped` and `satisfied` for a `//`-escaped path, a Windows-style or `..` root, and a `raw` that is no self glyph (handle, bare symbol, member glyph). Does it return a remedy only when the text is true, and `\"\"` otherwise?"
  - "Behavior change in `checkPathMissing`: the former `report` closure was inlined into `checkRef`. Confirm every call site still reports the same finding (`Check`, `Card`, `Ref`) and that the changed `Detail` format has no consumer, such as a test or stencil that matches the old string."
  - "Tests in `internal/planparser/validate_test.go`: the new subtest covers one `root:` and one glyph kind. Name any behavior the diff adds with no coverage, and check the test against the existing `TestValidate_PathMissing` subtests for duplicated coverage."
  - "Spec edits in `contracts/specs/loom-plan-checks-spec.md` and `contracts/specs/loom-plan-spec.md`: read them as the diff leaves them, against the code. Does the claim about worktree-relative self glyphs under `root:` match what `diskPathForRef` and the parser do? Do the repository's constraints document and the docs it names for this kind of change ask for more, such as a PATTERN entry or another spec?"
  - "Commit message claim: it says both the plan gate and Webster refuse the glyph with `path-missing` and `glyph-not-found`. Check the diff bears out the plan-gate half and does not change `glyph-not-found` behavior."
---

The decision record carries only the entry under `## Goal`, so the board entry is the measuring stick.
The diff is small: one commit that adds a remedy sentence to `path-missing`, a test, and two spec lines.
The main risk is scope: the entry asks for a shared parser and a bouncer that rejects what Webster rejects, and the diff may deliver a message improvement only.
