```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: rerun F1's search for the sites that state the bug-record rule, and check the fix covers each.

**Findings**

- **LOW** — `contracts/specs/loom-discussion-spec.md:124`, the rewritten opening line of `### A bug entry's record`. The line is 122 characters, over the 120-character limit that `prose` sets for body lines. It has no independent-clause break point, so it needs a rewrite. For example: "A `bug`-labelled entry's record, as the Discussion-Write lead writes it, states up front what the red-first gate later checks:". That is about the same length. A shorter form: "The Discussion-Write lead's record for a `bug` entry states up front what the red-first gate later checks:" (about 106 characters). Evidence: `awk '{print length($0)}'` on line 124 returned 122.

**Set check for F1**

The set is every site that states the bug-entry record rule. The search `grep -rn 'labelled `bug`\|bug-labelled' contracts/ internal/` found:
- `loom-discussion-spec.md:124` is the changed line.
- `loom-rubric-discussion-review.md:48` reads "For a board entry labelled `bug`, the record states…". It is a Discussion review rubric, so it applies only to Discussion-written records. The review already judged it consistent, and no change is needed.
- `loom-template-discussion.md:102` reads "When the entry is labelled `bug`, the design also settles…". It is the Discussion-Write stencil, so it is already scoped to that writer.
- `darn-template-write.md:107` and `internal/shedrecipe/doc.go:206` concern the red-first gate and darn's Record-Write. They are not the Discussion record rule, and the review did not name them.

No other site states the rule unscoped for Record-Write records.

**Unchecked:** none.

**Tool calls**
1. `grep -rn` for `labelled `bug`` and `bug-labelled` in `contracts/` and `internal/`, to list every site stating the rule.
2. `grep` and `git status` on the spec, to confirm the edit landed and the tree is clean.
3. `awk` length of spec line 124, to check the 120-character limit.
