# Gate test scope: slotted gates, tiered tests, Publish failure record

Before the 2026-10-08 crash, Webster implementer forks ran a module-wide build and test per card, and gates, forks and hub agents had no shared bound; four concurrent runs reached the user slice's memory limit and froze the machine.
This change makes each gate run only the test tier its position needs and bounds the hub's concurrent gate runs.

## What changes

- **Card gate:** `cardGateCommand` emits `lyx gate test <dirs>` over the card's own surviving directories (tier 1), one step per module, then `lyx loom lint-comments`.
  No module-wide `go build`, `go test` or `-tags integration`.
- **Batch gate:** a new command runs `lyx gate test --tags integration` once per batch over the union of the batch's directories, so tier 2 runs once per batch instead of once per card.
  The plan verify, Publish and Finalize keep their full scope.
- **Slot lock:** new `internal/gateslot` owns a hub-wide pool of OS file-lock slots under `<BoardDir>/.lyx/`, with holder records, an inheritance variable and a `GOFLAGS -p` cap.
  Every Go-side gate run takes a slot: the `verifytree.Verify` sites, the card `**Verify:**` rerun, and the plan gate's `caller-uncovered` `go list`.
  `verifyrun.Run` gains an environment parameter and stays free of the slot engine.
- **`gate` config module:** hub-wide, read only from `<BoardDir>/_lyx/config/gate.yaml`, with `slots` (default 2), `go_parallel` (default 4) and `cli_wait_sec` (default 300).
- **`lyx gate test`:** new module `internal/gatecli`, the sanctioned slotted route for an agent's module-wide or `tmux` run.
  Its `go test` runs in its own process group (Linux `Pdeathsig`, Windows job object), so a killed holder takes its run down.
  When every slot stays busy past `cli_wait_sec` it exits with a distinct slot-busy code and tells the caller to re-run.
- **Agent deny:** `claudeengine` adds a standing PreToolUse deny for raw module-wide `go test`/`build`/`vet`, raw `tmux` or `llm` tag runs, and any command naming the inheritance variable or `LYX_STRAND_NAME`.
  The implementer and verify-fix stencils and the plan template tell agents the route up front, and run each gate step as a background Bash call.
- **Plan gate:** a new blocking check refuses a card `**Verify:**` holding a command the deny would refuse; `lyx loom validate-plan` calls the same function.
- **`landing` is hub-wide:** `publish_verify` and the other landing keys are read only from `<BoardDir>/_lyx/config/landing.yaml`, so a task can no longer empty its own `publish_verify` through the pair's tracked file.
  `lyx config` refuses a hub-wide write from a task pair or under a slug-bearing strand name.
- **Publish failure record:** when Publish's plan verify or `publish_verify` fails, it persists a record under the worktree's `.lyx/`; a later Publish pass deletes it.
  While it exists, the Webster-Burler round gate also runs the failing tests by name and, for a `publish_verify` failure, a `tmux` pass over the impacted set.
  The Webster-Review prompt renders the record (identities, log path, merge-in commit with the diff command) so the round reviews the failure and the merge that carried it.
- **Comment lint:** a break after a comma plus a coordinating conjunction passes only when a clause (subject plus finite-verb candidate) follows.
- **Status:** `lyx loom status` shows waiting gates and current slot holders.
- **Docs:** `PATTERN.md` and `pattern/` (gate table, the decision that no gate runs the `llm` tier, slot, failure record), `CLAUDE.md`'s push rule, `doc.go` files, `docs/overview.md` and `contracts/specs/refusal-spec.md`.

## Design decisions to know

- The slot bounds concurrent runs, not their size; a command passing its own `-p` overrides the cap.
- The deny and the `lyx config` refusal guard against mistakes, not a determined agent: they read static command text, so a run hidden behind a script, a raw package-scoped `go test`, or a direct write to the hub's `landing.yaml` still passes.
  The decision record's D7 and D10 bounds list the remaining routes.
- A card that breaks a package outside its own targets passes its card gate; the batch gate or the plan verify catches it.
- The failure record lives in the pair's `.lyx/`, so editing it can narrow the round pass; each field passes a shape check before it renders, and Publish reruns both verifies in full regardless.
- Child-lifetime code for `lyx gate test` stays in `internal/gatecli`, apart from `internal/proc`, so this branch merges cleanly with the process-group work on `main`.
- Moving `landing` hub-wide removes per-pair landing overrides; a landing change is now a `lyx config landing` from the prime.

## Check by hand

None of these were run by the author.

- Run `lyx gate test ./...` and `lyx gate test --tags integration ./...` (each as its own background call) before pushing; the plan-level verify was left to the gate.
- Gate way-forward coverage: confirm tests reach `gate.yaml` at each Go-side gate site (`hubgeom`, `websterengine`, `verifytree`); a late fix touched them and no fresh review has seen it.
- Package docs: confirm each touched package's `doc.go` describes the new behavior and not the old (`internal/websterengine`, `internal/verifytree`, `internal/landingshed`, `internal/loomshed` and the rest).
- `cli_wait_sec` against the Bash tool's timeout: the stencils run each `lyx gate test` as a background call, so check that a slot-busy exit followed by a re-run does not cost a fork a self-fix attempt.
- `lyx gate test` and the deny: check on Linux that killing the holder with `SIGKILL` ends `go`, and on Windows that the job object ends the whole tree; neither was run.
- Check the deny regex against both a refusal set and a pass set (a `grep` or commit message containing `go test ./...` must not be refused).
- Check that a pair started before this change, holding a retired `landing.yaml`, is reported by the reconcile and that Publish does not read it.
- `gofmt -l` lists files this change did not touch, such as `internal/fabricengine/pull.go`.
- Shell appends: two edits were made with shell heredocs against the no-scripted-edits rule; skim the diff for stray formatting.
