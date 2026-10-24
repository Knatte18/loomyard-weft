```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 changes agree with the rest of the decision record and support log (fix-consistency lens).

The fix created four inconsistencies with nearby text. I found none in the other changes.

1. **MEDIUM.** D11's new start-guard sentence, and the matching D11 criterion: "passes batten's start guard and reaches Worktree-Create's skip".
   - The fix exempts the landing only from the left-pair refusal. It does not touch the tracked-dirt refusal that runs before it.
   - In `internal/battencli/startguard.go:67-76`, `(!found || atWorktreeCreate) && g.SourceDirty != nil` refuses a run with no status file over tracked dirt in the source worktree. This check runs before the `PairParts` check.
   - So a beam landing over a dirty prime still never reaches Worktree-Create.
   - D7's line "keeps its tracked-dirt refusal for a pair forked from the prime, and checks none for one forked from a beam" does not say what happens for a landing, which forks no pair.
   - The new criterion claims the start guard passes without that condition.
2. **LOW.** D5's new line "With `Parent` named, Add checks no tracked dirt at all". The line just above it, which was left as is, says "the dirty paths are logged at Info". Logging dirty paths means reading them, so "checks no tracked dirt at all" conflicts with it. It should read as checking without refusing.
3. **LOW.** D1's new sentence "an open project lists ... under a layer of its own, `Project`, never Ready or a dependency letter".
   - It does not say how a project held by its landing run lists. `ComputeLayers` puts a run-held entry under Running (`layer.go` doc).
   - D2's "a project whose status is a run status" and D15's standing "its Running status" both assume a run status can show.
   - Read literally, the new sentence lists a landing beam as `Project`, not Running.
4. **LOW.** D3's rewritten Bound: "D9 admits that create on the machine the project's effective owner names, and elsewhere refuses it naming the owner and `lyx adopt <slug>`". It has two problems:
   - D9's own text, which was not changed, still admits "a project ... whose `beam` is true and which this machine or no machine owns". It says nothing about the effective owner. It also gives a fuller way forward than D3's paraphrase: `lyx beam create <slug>` run on the owner's machine, or adopt and then create here.
   - The same D3 sentence uses `<beam>` for the create and `<slug>` for the adopt, two placeholders for the same entry.

No further tool calls were made.
