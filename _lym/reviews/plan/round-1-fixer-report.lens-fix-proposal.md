```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged the proposals that were carried out as written (F1, F2, F3). I made no edits.

**LOW**, about F2's proposal as carried out in card 2 (`followCmd` added to Edit, the Intent line on the follow help, the ImpactSummary clause).

- **Summary:** F2 presents its four additions as the rest of the one-overview comment sweep, but one more caller comment still says "else the overview". `Engine.AttachArgv`'s doc comment is in neither card 2's Edit list nor its Intent, so after card 2 it still states the old fallback.
- **Evidence:** `internal/reedengine/attach.go:43` reads "the newest live agent window, else the overview, never the shell". It sits on `func (e *Engine) AttachArgv` at `attach.go:50`. Card 2's caller-comment bullet lists only `FollowRun`, `selectRelevantWindowIfCurrentLocked`, `reconcileLocked` and `killStrandPanes`.
- **Also stale:** the comment above `TestAttachArgv_SelectsTheRelevantWindow` (`attach_test.go:359`) says the same. That test is already a card 2 target, but no Intent line rewrites its comment.
- **Already covered:** the other hits are on symbols card 2 already edits and names in its Intent: `relevantWindow` (`ownwindow.go:453`), `killStrandPanes` (`strand.go:835`), `reconcileLocked` (`reconcile.go:218`), `FollowRun` (`follow.go:30`) and the doc.go focus-rule and `FollowRun` paragraphs.

I found no concrete case against F1's proposal or F3's.
- **F1:** the orch entry's later "add, replace or record" still holds, because a stale runs strand is still replaced.
- **F3:** the IfAbsent relaunch runs the stored `Cmd` `exit` (`strand.go:629-631`). Under remain-on-exit the pane dies and the window stays, so the order assertion is still reached.

Tool calls:
1. grep for "else the overview" across the .go and .md files outside `_lym`: my context had only the four sites the review named, not the full set of fallback wordings.
2. grep for the functions that hold `strand.go:835`, `attach.go:43` and `attach_test.go:359`: my context did not show which symbols those comments belong to.
