# Card 4 — notifier-asker-walk

**Create:**
- `plan:internal/orchcli#NotifyAsker` -> `func NotifyAsker(location *lymcwd.Location, program, line string) (posted bool, err error)`
- `plan:internal/orchcli#askerOrchStrand` -> `func askerOrchStrand(location *lymcwd.Location, live func(worktree *lymcwd.Location) (bool, error)) (name string, recorded bool, err error)`

**Edit:**
- `internal/orchcli#TestParentOrchStrand`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/loomcli#loomCLI.wire`
- `internal/webstercli#websterCLI.wireHub`

**Extends:**
- `internal/orchcli/doc.go`

**Uses:**
- `plan:internal/orchroute#Asker`
- `internal/orchcli#postNotice`
- `internal/orchcli#OrchLive`
- `internal/burlercli/doc.go`
- `internal/loomcli/doc.go`
- `internal/webstercli/doc.go`

**Intent:**
Every hold-notice notifier wiring resolves its orch with the rule `lym creel post --parent` uses (D3), so the orch a hold notice reaches is the one the held agent's own question reached.
`NotifyAsker` sends line as creel mail, signed as the module's program, to the orch `orchroute.Asker(location, OrchLive)` answers: the walk from the pair's parent from a pair, the walk starting at the worktree itself from a beam or the prime.
It goes through `postNotice` like `NotifyParent`, so an unrecorded destination logs and posts nothing, and a refusal comes back carrying the full line.
The `SetNotifier` closures in `burlerCLI.wireHub`, `loomCLI.wire` and `websterCLI.wireHub` call `NotifyAsker` in place of `NotifyParent`; loom's other notices, the hand-back and landing notices among them, are not hold notices and keep their walk.

Both `NotifyAsker` and card 2's `--parent` resolve through the one `orchroute.Asker`, so for one held agent, in a pair and in a beam, the `--parent` receiver and the hold notice's orch are the same strand.
`NotifyAsker` takes its destination from a new unexported `askerOrchStrand(location, live)`, which answers `orchroute.Asker(location, live)`, and `TestParentOrchStrand` gains a row calling it from the beam `b` with `b`'s orch live and expecting `b`'s orch: the one case where it differs from `parentOrchStrand`, so a `NotifyAsker` routed through the old walk fails.
The three `SetNotifier` closures are one-line wiring that review checks.
`internal/orchcli/doc.go` adds `NotifyAsker` to the list of notice routes, naming it the hold notice's route.

**ImpactSummary:** Three notifier closures switch from NotifyParent to the new NotifyAsker; a hold notice from a beam run now walks from the beam itself, every other notice is unchanged.
