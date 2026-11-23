```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens, checking that the fix's changes agree with their sibling docs, tests and cross-references.

- **LOW** — `internal/gateslot/redfirst.go:15-16`, the `RedFirstUnjudgedExit` doc comment.
  - **Summary:** it lists the causes of exit 5 and was not updated.
  - **Evidence:** it names a non-go/dotnet scoper, and a C# change whose tests "no filter selects by name or no tier of the gate key can run". This round added two causes: a tier run that discovered no test (the filter selected none), and a restore or build failure that ended the run before any test ran. `internal/gatecli/doc.go` and the `lym gate red-first` help text (`internal/gatecli/redfirst.go:77-79`) were updated for the new causes, so this twin now disagrees with them.
  - **Suggested fix:** end the C# clause with "or whose tier run discovered no test".

- **LOW** — `internal/gateslot/doc.go:117`, "any other code is a refusal or a run that judged none of the tests".
  - **Summary:** it conflicts with the narrowed exit-1 meaning.
  - **Evidence:** exit 1 now means a run that started tests and judged none of them. A run that ended before any test ran now exits 5 instead. The `gatecli` doc and help were reworded to "ran tests and judged none", and this twin was not.

I found no other stale twins. These sibling files read consistent with the change:
- `internal/loomshed/redfirstgate.go:41-51`
- `internal/loomshed/doc.go:69`
- `internal/shedrecipe/doc.go:65`
- `contracts/stencils/darn/darn-template-write.md:113`

They name only the non-go/dotnet-scoper case, and the loomshed text ("a timeout or a runner crash") still holds.

Tool calls:
1. Grepped `RedFirstUnjudgedExit` and the exit-code wording across `.go` and `.md` files, to find sibling docs my context lacked.
2. Grepped those siblings for "judged none" and "neither" with line numbers, and read `internal/gateslot/redfirst.go:10-20`, to confirm the wording each one holds.
