# Card 9 — batten-owner-check

**Edit:**
- `internal/battencli/cli.go`
- `internal/battencli#battenCLI.armAt`
- `internal/battencli#battenCLI.reconcileResume`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#resumeRefusal`
- `internal/battencli#battenCLI.battenPreStepUnmarked`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.battenStatusExtras`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/battencli#TestBattenPreStep_RefusalKinds`
- `internal/battencli#TestRunInWindow`
- `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`
- `internal/battenshed/doc.go`

**Create:**
- `plan:internal/battencli#ownerSeams` -> `type ownerSeams struct`
- `plan:internal/battencli#hubOwnerSeams` -> `func hubOwnerSeams(location *lyxcwd.Location) ownerSeams`
- `plan:internal/battencli#battenCLI.refuseStartOwnedElsewhere` -> `func (c *battenCLI) refuseStartOwnedElsewhere() error`
- `plan:internal/battencli#battenCLI.claimPrimeOwner` -> `func (c *battenCLI) claimPrimeOwner() error`
- `plan:internal/battencli#battenCLI.ownerPreflight` -> `func (c *battenCLI) ownerPreflight(found bool, status shedengine.Status) error`
- `plan:internal/battencli#ownedElsewhereRefusal` -> `func ownedElsewhereRefusal(slug, owner string) error`
- `plan:internal/battencli#TestRefuseStartOwnedElsewhere` -> `func TestRefuseStartOwnedElsewhere(t *testing.T)`
- `plan:internal/battencli#TestBattenStatus_CarriesTheOwner` -> `func TestBattenStatus_CarriesTheOwner(t *testing.T)`

**Uses:**
- `internal/battencli#newFakeReceiver`
- `internal/battencli#battenCLI.checkRunStatus`
- `internal/boardengine#SetOwners`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#Owners.Effective`
- `internal/boardengine#OwnedElsewhere`
- `internal/boardengine#ErrOwnerOriginUnreachable`
- `plan:internal/fabricengine#EntryOwnedElsewhere`
- `internal/machinename#Name`
- `internal/fabricengine#ParentSlug`

**Intent:**
D3 for `lyx batten run` and `lyx batten step`, D2's first-act prime claim at batten and D9 for `lyx batten status`.
`ownerSeams` carries `Machine`, `Pull` (brings Bolt level through a pull-first write of nothing: a `fetch_failed` skip wraps `boardengine.ErrOwnerOriginUnreachable`, and a `diverged` or `dirty` skip refuses naming its detail, which carries its way forward), `Read` (`boardengine.ReadOwners` of the local Bolt), `ParentOf` and `Set` (`boardengine.SetOwners`); `battenCLI` (in `cli.go`) gains an `owner ownerSeams` field, which `Command` and `armAt` fill with `hubOwnerSeams` when it is zero, so every armed path, `ArmAt` for `lyx shed run` and `Arm` included, carries the check, and a hand-populated receiver that calls no arm leaves it zero, reading as no owner check.
`ownedElsewhereRefusal` words the one refusal: "battencli: %q is owned by machine %q, so this machine does not start, resume or land it; way forward: run `lyx adopt %s` from this machine's prime, which takes it over and resumes its run, or records the owner when no run exists here, after which `lyx batten run %s` starts it".
`ownerPreflight` is the owner check of a run about to be driven, shared by `battenPreRun`, `battenPreStepUnmarked` and `runInWindow`: with no status file, or a status still at Worktree-Create whose pair this machine does not hold, it is the start check, `refuseStartOwnedElsewhere`; otherwise it is `reconcileResume`.
So a run whose Worktree-Create row was refused before the pair existed resumes through the start check and never meets the run-scope reconcile's "no pair worktree" error.
Start: `armAt` calls `refuseStartOwnedElsewhere` for the `run` and `step` verbs when the slug's batten status file is absent, before `armSeed` writes a seed and before anything is claimed.
It pulls Bolt, and an unreachable origin refuses ("a start records this machine as the task's owner, which needs origin; re-run `lyx batten run <slug>` once origin answers"); it claims the prime through `claimPrimeOwner` when the local Bolt records no prime owner (a write the replay lost leaves the other machine's claim, which the next read takes); it then reads the entry's effective owner from the Bolt it just pulled, and an owner elsewhere refuses through `ownedElsewhereRefusal`; a slug with no board entry has no owner and passes.
Claim: the `CreateWorktree` closure in `wire` replaces its warn-only `SetRunStatus` with `c.owner.Set` carrying this machine, the slug and the `running · Worktree-Create` claim in one Bolt commit, without `Takeover`, and returns the setter's error, so a failed owner write or push refuses the Worktree-Create row and the stopped run resumes with the same verb; a slug the setter reports in `Lost`, because the replay kept origin's owner or the pull found the entry owned elsewhere, refuses through `ownedElsewhereRefusal` with that owner, so the claim never takes a run from another machine; an absent entry writes nothing and the row goes on, as today.
The inline comment in the closure ("A board failure only warns") is replaced to match.
Resume: on a reconcile that reached origin, `reconcileResume` claims the prime when none is recorded; a failed claim, an unreachable push, a refused push or a diverged or dirty pull, warns naming its cause and goes on to the owner verdict, as orch start's claim does; with origin unreachable the resume makes no owner write and keeps D3's Warn and resume.
Then `resumeRefusal` gains the owned_elsewhere case, refusing through `ownedElsewhereRefusal` with the entry's owner.
An active entry that is run-held with no owner recorded gets its owner through `Set` before the resume; an unreachable origin then refuses naming the re-run once origin answers, a refused push refuses with the push's class and its way forward, and a lost write refuses through `ownedElsewhereRefusal`.
With origin unreachable at the reconcile, the resume reads the local Bolt: an owner elsewhere refuses ("the local board shows %q owned by %q; re-run `lyx batten run %s` once origin answers"), and a task owned here, or run-held with no owner recorded, keeps today's Warn and resumes with no owner write.
Bound: a resume with origin unreachable can drive a run adopted elsewhere since this machine's last fetch, until D6's next check or Finalize's landing reconcile catches it.
Step: `battenPreStepUnmarked` runs `ownerPreflight` for a found status after its done and state checks, and any refusal there is `shedverbs.KindBootstrap`, so `lyx batten step` drives no run owned elsewhere; for an absent status `armAt`'s start check has already run.
Window: `runInWindow` runs `ownerPreflight` after `checkRunStatus` and before it opens the window, so issue #741's `lyx batten run <slug> --window` refuses on the envelope, naming the owner and `lyx adopt <slug>`, instead of inside a window.
`battenStatusExtras` adds `owner_machine`, the slug's effective owner on the local Bolt, and `owned_here`; an unreadable board adds neither and logs one Warn.
The package doc comment atop `cli.go` replaces the warn-only claim paragraph with the owner claim and the refusals, and `battenshed`'s doc's resume paragraph gains the owned_elsewhere refusal, the owner write and the unreachable rule.
`TestRunCmd_ResumeDispositions` gains rows with fake owner seams: an owned_elsewhere verdict refuses naming the owner and `lyx adopt some-slug` and runs nothing; an active verdict on a run-held unowned entry writes this machine before resuming; that write returning `ErrOwnerOriginUnreachable` refuses naming the re-run; that write returning a refused push refuses naming it; a lost write refuses naming the other owner; an unreachable reconcile with the local Bolt owned elsewhere refuses naming the re-run; an unreachable reconcile on an unowned run-held entry warns and resumes with no write; a hub with no prime owner records this machine as the prime owner; an unreachable reconcile on a hub with no prime owner warns and resumes with no write; a failed prime claim on a reachable reconcile warns and still reaches the owner verdict; a status at Worktree-Create with no pair runs the start check and no reconcile.
`TestBattenPreStep_RefusalKinds` gains a row: a found running status whose reconcile reads owned_elsewhere is `KindBootstrap`, naming the owner.
`TestRunInWindow` gains a row: an owned_elsewhere verdict refuses and opens no window.
`TestWire_CreateWorktreeClaimsTheBoardEntry` replaces its warn-only row with rows over a fake `Set`: a setter error fails the row naming the cause; a slug in `Lost` fails the row naming the other owner; an absent entry passes with nothing written; and the claim row, under `BOARD_SKIP_GIT` over `boardkit.HubWithBoardConfig`, records `owner_machine` beside the claim status through the real setter.
`TestRefuseStartOwnedElsewhere` is a table over `refuseStartOwnedElsewhere`: an entry owned elsewhere refuses, an entry owned here passes, an absent entry passes, an unreachable pull refuses naming the re-run, a diverged pull refuses naming its detail, an empty machine name refuses naming the hostname cause, a hub with no prime owner claims it for this machine, and a lost prime claim reads the other machine's claim; one more row drives `armAt` for `run` over an entry owned elsewhere and asserts no seed file was written.
`TestBattenStatus_CarriesTheOwner` asserts `owner_machine` and `owned_here` on a status envelope for an owned-here and an owned-elsewhere slug.

**ImpactSummary:** battencli's arm, pre-run, step hook, window path, claim row and status envelope; no other package changes, the shed resume verbs follow in card 10.

**Commit:** 9: batten: start, resume and claim only what this machine owns
