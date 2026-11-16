```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F1, F2 and F3, which the fixer carried out as proposed.

No findings.

- **F1**
  - **Constant collision:** `RemoteControlOnVerb` and `RemoteControlLit` are both `"rc-on"`.
  - **Why it's harmless:** the bar's `m:*rc-on*` match reads the state job's printed output, never the composed click line. The click line therefore can't be mistaken for a state word.
  - **Verb names:** the cobra `Use` strings and the composed lines now share one declaration. `cli.go:153` and `remotecontrol_test.go` still resolve, because `remoteControlBarVerb` stays as an alias of the reedengine constant.
- **F3**
  - **Moved assertion:** it runs for every row that reaches the success path. The `withdrawErr` row returns early, so it never reaches it.
  - **Rows without a seeded record:** they now expect `RemoteControlUnlit`. Deleting the `recordUnrecordedRemoteControl` call would leave the word empty, so those rows fail.
  - **New row:** it seeds `{Strand: "g1", On: true}` on an already-running strand. An unconditional off-write in place of the keep-the-record path would flip it to unlit, so it fails too.
- **F2**
  - **Summary edits:** the new bullets match `turnOnRemoteControl` (refuses on a lit record, way forward names `/remote-control` typed by hand). They also match `recordRemoteControl` (no record on an engine without `RemoteController`) and `remoteControlBarWordAt` (needs a record naming the strand).
  - **Wording check:** I found no remaining sentence in the Design-decisions, Open-risks or Tests sections that contradicts them.

Tool calls made: none, because my context already held every changed file.
