```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the fix round's change set.

Findings:

- **LOW** — `contracts/specs/loom-final-summary-spec.md:44-45` was left as it was. It is the contract for the Merge danger section ("The section runs to the next heading of level one or two. Either line may stand as a list item…"). F1 changed the scan in `internal/summaryparser/description.go` (`mergeDangerSection`, `isFenceLine`) so that fenced lines are neither the heading nor a section end, and the spec does not say so. The spec itself shows the section inside a fence at lines 31-36, which is the shape F1 targets. A reader of the spec cannot tell that a fenced example is ignored. A sentence such as "Lines inside a fenced block are not read" would fix it.

- **LOW** — `isFenceLine` in `description.go` toggles on any line starting with ` ``` ` or `~~~`. It ignores fence length and type, so a ` ``` ` line inside a `~~~` block closes it. `contracts/specs/loom-discussion-spec.md:63-64` already states that a fenced block follows CommonMark's top-level fence rule. Two fence rules in one repo can drift on edge cases. The case is rare and F1's fix text asked for no more, so it only needs a mention.

Checked with no finding:
- The stencils `landing-template-describe.md:56` and `darn-template-write.md:73` only say to close with the section the spec defines, so they are unaffected.
- The `contracts/orchrole/role-beams.md:32-36` description of `--draft` omits the placeholders. It never listed the draft's sections, so it is not stale.
- The `cli.go` help, the `doc.go` paragraph and the `describe.go` `way_forward` now agree with each other.
- The `way_forward` string has no test pinning it. The grep found the string only in `describe.go`.

Calls made:
1. `grep` over `*.md` and `*.go` outside `_lym`/`.lym` for other `beam describe --draft` text, other `Merge danger` mentions and fence rules. My context lacked the other docs and specs.
2. `grep`/`sed` on the final-summary spec, the two stencils and `role-beams.md`, to see whether they restate the section or draft contents.
3. Read `contracts/specs/loom-final-summary-spec.md:20-49` to see the spec's section rules in full.
