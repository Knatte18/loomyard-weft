```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that each claim the fix added or rewrote holds against the code at HEAD.

**1. MEDIUM. D2, the new sentences "This turns `adoptcli`'s `parentWarning`, which already judges every entry this way, into the refusal." and "The test is made against the entries the call records an owner for".**
Summary: `parentWarning` runs only after the owner write, so turning it into the refusal would refuse once the adopted entry is already owned here. That leaves the same split shape the refusal exists to prevent.
Evidence:
- `internal/adoptcli/adopt.go:161`: `deps.SetOwners(...)` records the owners first.
- Lines 183-189 then read `after, err := deps.ReadOwners()` and call `resumeSlug(deps, here, after, ...)`.
- Only inside that call does `parentWarning(deps, here, owners, slug)` run (`adopt.go:355`).
- So "the entries the call records an owner for" are known only after the write.
- The text does not say the check must run before `SetOwners`. To do that, it would test against `adoptable` plus the prime or parent named, and treat a parent the write reports in `write.Lost` as not adopted.
- As written, the claim says the existing warning site becomes the refusal, and that site cannot refuse before the write.

**2. LOW. D5, the new sentence "a note or project turned into a task by an upsert or `lyx board promote`".**
Summary: the sentence reads as if `lyx board promote` can turn a project into a task, but `Promote` refuses a project.
Evidence: `internal/boardengine/store.go:719-720`: `if current.Kind == KindProject { return ... "a project is never made a task" }`. Only an upsert changes a project's kind.

Every other claim I checked holds:
- `refuseCovered` runs first in `battenPreRun` (`arm.go:644`).
- The hand-pass refusal names `lyx resume` (`pauserecord.CoveredError`).
- `ResumeInWindow`'s callers are `beamcli` and `adoptcli` (`beamcli/deps.go:115`, `adoptcli/deps.go:92`).
- `refuseResumeUnreachable` judges the owner against the local Bolt (`battencli/owner.go:258`).
- `set-status` refuses an absent `status` key so a typo cannot clear it (`boardcli/cli.go:642`).

Tool calls:
- `grep` for `ResumeInWindow` callers, `refuseResumeUnreachable` and adopt's step comments: the code behind the new F2 and F4 claims was not in my context.
- Read `adopt.go` lines 180-300 (grep) and `owner.go` lines 255-290: the adopt flow's order and the unreachable refusal's body were not in my context.
- Read `adopt.go` lines 150-195: needed where `SetOwners` sits relative to `parentWarning`.
