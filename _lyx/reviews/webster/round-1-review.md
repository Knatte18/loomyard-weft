---
verdict: APPROVED
---

No defects found in the committed range (`git merge-base main HEAD..HEAD`, ten commits, cards 1 to 10).
`product.parent` is `main`, and no `_lyx/loom/rework/` directory exists, so the range is the merge-base diff.
`lyx gate test` over the touched packages (`shuttleengine`, `shuttlecli`, `burlercli`, `twilldirective`, `loomengine`, `shedrecipe`, `contracts/stencils`, `loomcli`, `burlerengine`, `battencli`) passes.

## Focus verdicts

- **Card 1, shared poll.** `awaitAny` in `internal/shuttleengine/await.go` runs over the `Clock` seam (`clock.Now`, `clock.Sleep(min(pollFloor, remaining))`), and `AwaitAny` is the `realClock` entry.
  `await-review` keeps its single argument, output keys, `--cap` default (`DefaultAwaitCap`, 250 s) and the stat-error exit 1.
  The poll moves from 200 ms to the one-second floor, as the card states.
- **Card 2, `lyx shuttle await`.** With several paths, the first existing one in the order told ends the wait and is returned under `path`.
  On the cap it prints ok with `ready: false`, `watched` and the message, exiting 0.
  A stat error or a cancelled context is a JSON error with exit 1.
  `Command`'s pre-run skips resolution for `awaitVerb`.
- **Cards 4 and 5, host neutrality.** The two stencils and `internal/twilldirective` name no Discussion literal.
  `hostNeutralClaims` pins the absence of the host phrases in both stencils.
  Step names, the self-check command, the disposition place and the rubric path are built in `internal/loomengine/twill.go`.
- **Cards 8 and 10, ready marker.** `DiscussionReadyMarker` is built from `lyxdirs.DotLyxDirName` at the mirrored subpath.
  `RemoveDiscussionReadyMarker` is reached only through `Env.PrepareDiscussionSpawn`, which the `DiscussionWrite` entry runs inside the producer's fresh-spawn preparation.
  `SingleLLMProducer` runs that preparation only after the attach probe finds no live writer, so an attach or relaunch keeps the marker.
- **Cards 7 and 9, `twill`.** The spec differs from `single` only in the prompt and `ForkSubagents`.
  The unknown-value error names all three values, and `refusal-spec.md` carries the three refusal rows.
- **Card 6, template markers.** Each marker replaces a blank line, `setOffParagraph` returns empty for an empty block, and both names are optional in the composer and the chair table, so `single` and `seats` render the same bytes.
- **Test economy.** The new top-level tests (`TestAwaitAny`, `TestAwaitVerb`, `TestDiscussionSpec_Twill`, the three `twilldirective` tests) each cover a surface no existing test covers, and the budget rows match.
  The marker removal, the unknown-value error and `ForkSubagents` are each covered by an extended existing test.
