# Card 9 — darn-verify-prose

**Prosa:**
- `contracts/stencils/darn/darn-template-write.md`
- `pattern/PATTERN-verified-tree.md`
- `pattern/PATTERN-test-speed.md`
- `docs/overview.md`

**Intent:** Restate the texts D6 names, which this change makes false.

`darn-template-write.md`'s "What happens after you hand off" section stops saying the handoff runs the repository's full verify command, and says it runs a verify scoped to the diff on the committed tree.
`PATTERN-verified-tree.md` restates its darn sites: the Darn-Write and Darn-Burler-Fix gates run the tree-scoped command or the command `impactset` derives from the merge base through `Verify`, with `BaseCommand` set to `darn.yaml`'s `verify`, and the whole `verify` where `impactset` falls back; a darn run's Merge-In and Finalize run the tree-scoped command for a tree-scoped diff and the whole `verify` otherwise; and the `publish_verify` sentence adds that a darn Publish skips it for a diff tree-scoped at that call.
`PATTERN-test-speed.md`'s guard bound says an unmarked scanning test passes the round gate and darn's writer gates, is caught by the whole verify at the sites that keep it for a diff touching production code, and for a tree-scoped darn diff can reach the parent, bounded by what the hub lists as non-production in `darn.yaml`'s `tree_scope`.
`docs/overview.md`'s darn paragraph adds that the darn gates run the diff's impacted set or its trees' command, and that `tree_scope` and `tree_scope_verify` in `darn.yaml` scope a non-production diff at every site.
Every line follows semantic line breaks.
