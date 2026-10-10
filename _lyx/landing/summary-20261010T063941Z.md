# Gate test scope: slotted gates, tiered verification, failure record

Each gate now runs the test tier its position needs, and the hub never runs more gate builds and tests at once than a configured slot count allows.
Before this, Webster forks ran module-wide `go build ./... && go test ./...` plus `-tags integration` on every card, and with four runs in flight the machine ran out of memory.

## What changes

- **Card gate:** `cardGateCommand` emits `lyx gate test <dirs>` over the card's own surviving directories, one step per module (`-C <module>` for nested modules), then `lyx loom lint-comments`.
  There is no `go build ./...`, no module-wide test and no `-tags integration`.
- **Batch gate:** the implementer prompt gains one `lyx gate test --tags integration` pass per batch over the union of the batch's directories.
  Webster's gate, Publish and Finalize keep running the plan's `## verify:` in full.
- **Slot pool (`internal/gateslot`):** a hub-wide pool of OS file-lock slots under `<BoardDir>/.lyx/`, with holder records.
  Every Go-side gate run acquires a slot: `verifytree.Verify` sites, the card `**Verify:**` rerun and the plan gate's `caller-uncovered` `go list`.
  The spawned shell sees `-p=<cap>` appended to `GOFLAGS` and an inheritance variable, so a verify command that calls `lyx gate test` does not take a second slot.
  `verifytree.Verify`'s verify timeout starts once the slot is held.
- **`gate` config module:** hub-wide, strict, with `slots` (default 2), `go_parallel` (default 4) and `cli_wait_sec` (default 300).
  A value below 1 fails the load naming the key.
- **`lyx gate test`:** the one sanctioned route for an agent's slotted `go test`.
  It waits up to `cli_wait_sec`, then exits with a distinct code and a JSON error naming the holders; the way forward is to re-run.
  Its `go` child runs in its own process group (a kill-on-close job object on Windows, `Pdeathsig` on Linux), so a killed holder takes its run down and frees its slot.
  A `-C` target outside every hub runs unslotted under the template's `-p` cap.
- **Status:** `lyx loom status` shows each waiting gate with its wait start and the hub's slot holders.
- **Agent deny:** `claudeengine` refuses raw module-wide `go test`/`go build`/`go vet`, raw `tmux`/`llm`-tagged `go test`, and any command naming the slot-inheritance variable or `LYX_STRAND_NAME`.
  The plan gate and `lyx loom validate-plan` share a check refusing a card `**Verify:**` the deny would refuse.
- **Stencils and docs:** the implementer and verify-fix stencils name each fork's packages, the `lyx gate test` route, and run each gate step as a background Bash call; a slot-busy exit is a re-run, not a failure.
  `CLAUDE.md`'s push rule, `pattern/PATTERN-test-speed.md` (gate table, `llm` tier stays by hand), `PATTERN.md`, the touched `doc.go` files and `contracts/specs/refusal-spec.md` are updated.
- **`landing` is hub-wide:** `publish_verify` and every other landing key are read only from `<BoardDir>/_lyx/config/landing.yaml`, so a task cannot empty its own `publish_verify`.
  The existing hub-wide reconcile seeds the hub file from the prime's copy.
  `lyx config` refuses a write to any hub-wide module from a task pair, or under a slug-bearing `LYX_STRAND_NAME`; the hub orch and the prime keep their writes.
- **Publish failure record:** a failing plan verify or `publish_verify` persists a record under the worktree's `.lyx/` (kind, failing test identities, log path, HEAD, merge-in commit).
  While it is present, the Webster-Burler round gate also runs the failing identities by name and, for `publish_verify`, an impacted-set `-tags tmux` pass.
  The Webster-Review prompt renders the record so the reviewer also reads the merge that carried the defect.
  Every field passes a shape check before it renders or reaches a command.
- **Comment lint:** a new comma-plus-conjunction break passes only when the text after the conjunction opens with a subject and holds a finite-verb candidate.

## Design decisions to check

- The slot bounds concurrent gate runs, not their size; one run is capped only by `-p`, and a command that passes its own `-p` overrides the `GOFLAGS` cap.
- Polling gives no FIFO order, so a waiter can be overtaken; the wait is visible in status, not ordered.
- The deny and the plan-gate check see a command's static text only.
  A module-wide run hidden behind a script or a shell variable passes, and so does a raw package-scoped `go test`, which stays unslotted.
- The `lyx config` refusal guards against a mistaken write, not a determined agent; a direct file write to the hub's `landing.yaml` still passes.
- Landing keys can no longer be overridden per pair; a change is an operator or hub-orch `lyx config landing` from the prime, effective for every pair.
- The failure record sits in the pair's own `.lyx/`, so it can be deleted or edited.
  The shape checks bound what an edited record can inject, and Publish reruns both verifies in full regardless.
- `lyx gate test`'s child-lifetime code lives in `internal/gatecli` only, so it does not collide with the `internal/proc` work in `git-robustness`.
  `internal/verifyrun` keeps `main`'s `proc.ConfigureGroupKill` call.
- `CLAUDE.md`'s push rule keeps the line that exempts text-only commits from the test run.

## Check by hand

- Package docs: confirm the `doc.go` of the touched packages describes the new behavior and not the old (open finding `package-docs-describe-old-behavior`).
- Run `lyx gate test ./...` and `lyx gate test --tags integration ./...`, each as its own background Bash call; the plan-level verify was left to the gate and these were not run by hand.
- `gofmt -l` lists files this change did not touch, such as `internal/fabricengine/pull.go`; confirm none of this change's files appear.
- Review the files added with shell appends instead of edits for stray content; the run reported appends to a few files in two batches.
- Run `lyx gate test` against a held slot and a `SIGKILL` on Linux by hand to confirm the `go` child ends; the Windows job object is untested here.
