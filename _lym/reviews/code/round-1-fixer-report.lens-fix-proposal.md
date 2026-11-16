```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 (key constants) and F2 (doc wording) proposals as carried out.

**Findings**

- **LOW** — `internal/boardengine/front.go:57`, `[]string{"front", onLandKey}`: this list now mixes a constant with a literal, while the same file's `fields["front"]` and `delete(fields, "front")` stay literal. The `front` key has no constant, so the file now reads inconsistently. No behaviour breaks, and the review did not ask for a `front` constant. A reader will see one key named by constant and its sibling not.
- **LOW** — `internal/boardengine/store.go:500`, the `UpsertPayloadKeys` slice: it now names `pullRequestKey` and `onLandKey` as constants among about 25 bare literals, including `"notify_on_land"` and `"front"`. The constants cover only part of the key list, so the list is no longer uniform. This follows from the reviewer's narrow proposal, and the fix applied it as written.

**Checked, no breakage**

- The F1 constants are unexported strings. The sites in `replay.go`, `front.go` and `store.go` are all in package `boardengine`, so they compile (the gate passed).
- The remaining literals of these keys sit outside `boardengine`: `beamcli/mergeparent.go:79`, `fronttask/fronttask.go:100`, `dobbyengine/config.go`, and `hubissues`. They are in other packages and cannot use the unexported constants, and `dobbyengine`'s `pull_request` is a different config key.
- F2: `recipeCell` appends `recipeProfileSeparator` and the profile in the Recipe column, and `entryCell` never renders the profile. The new wording "The README's Recipe cell names only a profile the task carries" is therefore correct.

**Further tool calls (1)**

1. `grep` for `"on_land"` and `"pull_request"` in non-test Go files, `recipeCell`, and `git show HEAD --stat`: I needed to see whether other call sites stayed literal and whether the F2 claim holds, and my context did not show either.
