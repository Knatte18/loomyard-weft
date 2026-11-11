```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer proposals the fixer carried out as written (F1, F2, F3, F6, F7, F8, F9, F10, F11). F5 was fixed a different way, so it is outside this lens.

**[BLOCKING] F3's added-const exemption misses a const added inside an `iota` group**

- **Changed text:** `decision-record.md` D5, first bullet: "an added declaration no unchanged code can reach: an added const, …". The same claim is repeated in the D5 rationale ("A new plain function, const, … is reached only through changed code") and in the D5 criterion "an added const in A … is no miss".
- **Case:** A has `const ( KindA Kind = iota; KindB )`, and the edit inserts `KindNew` between them.
  - `KindB`'s text is unchanged, so it is not "a changed or removed const", but its value moves from 1 to 2.
  - Unchanged functions that compare against `KindB` keep their declaration text, so D4 marks nothing changed.
  - The added const is exempt, so D5 raises no miss.
  - B's tests that exercise `KindB` are not selected, and the edit narrows to `none selected`. Before the fix, any non-body change was a miss, so this edit used to widen.
- **Evidence:** `iota` groups are common in the most widely linked package. Examples:
  - `internal/fabricengine/reflog.go:21` `rewriteNone rewriteKind = iota`
  - `internal/fabricengine/midmerge.go:17` `MidMergeNone MidMergeKind = iota`
  - also `dirtiness.go:26`, `destroy.go:233` and `remoteleftover.go:28`
- **Same gap elsewhere:** any const spec whose value is implicit, repeating the previous expression, has it too. The Constraints count of "94" carries the same blind spot, because the scratch classifier compares const specs one by one.
- **Possible fix:** make an added, removed or reordered spec in a const group with an implicit value or `iota` a miss, or compare such a group as one declaration.

Tool calls beyond my context:
- grep for `= iota` in widely linked packages, plus `ls *.go` at the root: my context did not show whether `iota` const groups exist in those packages, which decides whether the case is real.
