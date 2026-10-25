```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the review proposals the fixer carried out as proposed.

**1. BLOCKING: card 39, live-pair way forward (F16, fixed as proposed)**
- Changed text: "for a live pair … the way forward names first … `lyx tender pause <slug>`, under whose pause entry `LivePair` reads not live, then the named call, then `lyx resume <slug>`". The new `TestAdopt` row also has the pause covering the parent so that `LivePair` reads not live.
- Summary: a slug pause does not make adopt's `LivePair` read not live, so the named call is still refused at step 1. F16's premise is left unresolved.
- Evidence: `internal/adoptcli/deps.go:45` sets `options.HubPaused = hubOnlyPaused(geometry.RecordPath)`. `internal/adoptcli/doc.go:19` says `hubOnlyPaused` "covers a slug only by a live hub entry, so a pair whose strands are alive under any slug-scope pause, an owner pause included, reads live". `lyx tender pause <slug>` writes a pause of the slug's scope, not a hub entry. The real answers at step 1 are `--keep-local`, whose step-1 arm passes a live pair (`internal/adoptcli/adopt.go` `takeOverSlug`), or a hub-wide pause.

**2. MEDIUM: card 15, failed-revive text (F17, fixed as proposed)**
- Changed text: `driver strand could not be revived; run "lyx loom resume" in the task worktree to bring a driver back`.
- Summary: this text goes out for a halted or awaiting child, where `lyx loom resume` does not bring a driver back.
- Evidence: `internal/loomcli/resume.go:233-241`. Resume "wakes a halted run's live, parked driver"; it "never spawns an LLM driver". It spawns a Go driver only for a halted go-seeded run, "which re-steps the run". You reach for it "after `lyx loom circling accept`", or when a run halted at a hand-back should go on.
- Case: an LLM-seeded child halted with a dead strand gets no driver from it. A go-seeded halted child is moved past its halt before the halt has been answered. The way forward for a failed revive of a halted child needs a step that brings a driver back without acting on the halt. For a halted LLM run that is `lyx loom start` or tender's revive, not resume.

**3. MEDIUM: card 37, `dobbyCmd`'s bound (F11, fixed as proposed)**
- Changed text: "`dobbyCmd` refuses a path `DobbyGeometry` cannot place as the prime or a beam pair of this hub". `TestDobbyCmd_RefusesAPathItCannotPlace` runs the verb "on a directory of the hub that is neither the prime nor a beam pair".
- Summary: card 27's `DobbyGeometry` takes a worktree name and refuses only "a worktree name with no directory". It never checks the beam mark.
- Case: a hand `lyx beam dobby <task pair root>` passes `DobbyGeometry`, takes that pair's lock and runs a dobby over a task worktree. The claimed bound does not hold, and the new test row fails against the card-27 geometry.
- Needed: the refusal has to read the beam mark from the pair's origin record, as `DobbyStart` does, or `DobbyGeometry` has to refuse a pair without the mark.

**4. MEDIUM: card 4, undecodable state rewritten fresh (F5, fixed as proposed)**
- Changed text: "`UpdateState` then writes a fresh record … card 32 reads that fresh record as a start, which owes every run a full pass".
- Case: the decode failure happens mid-run, during a tick's `UpdateState`. The fresh record drops several things:
  - **Tidy in flight:** the strand, report path and deadline go. Card 36's "one tidy agent at a time per dobby" breaks: the next tidy spawns a second agent, and the first is never stopped.
  - **Counted landings:** these go too. Every Land Done already in the records counts again, which forces a spurious tidy. Card 36's guard, "a dobby that starts with no state record counts every Land Done already there as seen", applies only at start.
  - **The dobby's pid:** card 31's `dobbyReading` then reads no live dobby, and the watched marker takes a hand-pass pid.
- The claim "card 32 reads that fresh record as a start" holds only when `Run` starts, since card 32 reads the pid at start only. A rewrite during a run owes no full pass.
- Needed: the fresh record must be treated as a start in the running loop too: re-record the pid, owe every run a full pass, and count the existing Land Done as seen. Card 36 must also say how it finds an in-flight tidy whose record was lost.

Tool calls:
- Read `internal/battencli/cli.go`'s goto help, `internal/adoptcli`'s `HubPaused` wiring and `internal/loomcli/resume.go`'s help. My context held none of the three: they were needed to test the F4 refusal's verb, F16's `LivePair` claim and F17's resume claim.
