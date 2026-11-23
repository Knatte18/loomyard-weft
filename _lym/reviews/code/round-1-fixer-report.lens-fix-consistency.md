```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for stale siblings and twins (lens fix-consistency).

**MEDIUM — `_lym/landing/summary.md`, O4 bullet: the new `plain_archive_tags_error` key is not mentioned.** The bullet still says only "`lym fabric cleanup --remote` reports them under `plain_archive_tags`".
- F4 changed the verb's contract: a failed listing is now reported under `plain_archive_tags_error` and the verb exits 0, where it used to fail.
- The fixer updated the summary for F5/F6/F10 (O1) and F9, but not for this behaviour change.
- Summary section "Merge danger" has the same gap. It says nothing about the envelope key that was added or the verb that no longer fails.

**LOW — `internal/fabricengine/doc.go:563`: the O1 text is stale.** It says `RefuseHubWithoutBolt` "refuses it naming the `--reset` clone that replaces it".
- For a one-repository hub the refusal now names two clones: the join, for an origin that holds the board, and the seed, for an origin that holds none.
- A lost-refspec store now carries the layout error (`ErrRecordsLayout`) in the refusal.
- The sentence after it ("`CloneHub` names the same re-clone") still holds only for the join form.
- `internal/fabricengine/doc.go` is the engine's contract doc and sits in the same package as the changed code.

**LOW — `internal/fabriccli/doc.go:26-27`: the same staleness.** It says "The refusal names the `--reset` clone that replaces the hub, as a plain re-clone into the same directory does." For a one-repository hub the two refusals no longer name the same text: reconcile gives join and seed, a re-clone gives the join only. The F7 test pins this difference.

**LOW — `internal/fabricengine/doc.go` near line 529: no mention of `ErrRecordsLayout`.**
- The doc describes the layout-read-first rule.
- It does not mention that the refusal now matches `ErrRecordsLayout`, or that battencli's catch-up relies on that match.
- Only `originNamespaceAt`'s own comment records it.
- This is a stale cross-reference a reader of the doc would notice, not a wrong statement.

No further call-site twin of the `fetchRunForCatchUp` change was checked by me beyond the one grep below. The grep found no other `fmt.Errorf("%w: %v", dobbyengine.ErrOriginUnreachable, ...)` wrap of `FetchPairTips`. The grep ran over `.go` and `.md` files only, and excluded `_test.go`.

Tool calls:
1. `grep` for `plain_archive_tags` / `RefuseHubWithoutBolt` / "no repository at its" in non-test `.go` and `.md` files, to find sibling docs and cross-references my context lacked.
2. `git diff main...HEAD -- internal/fabricengine/doc.go`, because my context did not show the engine doc's changed lines.
