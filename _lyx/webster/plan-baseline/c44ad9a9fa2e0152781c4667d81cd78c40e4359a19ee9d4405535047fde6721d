# Card 11 — batten-run-window

**Edit:**
- `internal/battencli/cli.go`
- `internal/battencli/cli_test.go`
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/claims_test.go`
- `internal/battenshed/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/reedengine/window.go`
- `internal/hubgeom/hubgeom.go`

**Intent:**
D7, CLI side: `lyx batten run <slug> --window` starts the same batten run in its own tmux window of the orch's reed session and returns at once.

`cli.go` adds a `--window` bool flag to the run verb only.
With it set, the run verb's body is replaced by one that, after the usual prime resolution and slug checks of `resolvePersistentPreRun`, opens the window through card 10's reed op on the prime's reed engine (`reedengine.LoadConfig` and `hubgeom.ReedGeometry` of the prime location), named `batten:<slug>`, running `lyx batten run <slug>` (without `--window`, carrying an explicitly typed `--driver`/`--child-driver` through), and prints a JSON envelope naming the window id and name and whether it found one already live ("already running; nothing started").
The envelope reports a started window only, never a started batten: a second batten for the slug is refused by the run's own lock inside the window and read from batten's own log.
The window opener is a seam on `battenCLI` so the refusals are tested without tmux.
Every refusal names its way forward (`PATTERN-refusal-way-forward`; `refusal-spec.md` lists batten refusals out of scope, so no spec row lands):
- no reed session for the prime: `lyx orch start`, or plain `lyx batten run <slug>` in a terminal;
- a multiplexer without `new-window` (the reed capability error) and a failed `new-window`: `lyx batten run <slug>` in a terminal;
- the read-back sentinel: `lyx batten status <slug>`, since the run may already be live from the call that kept the older window, and `lyx batten run <slug>` in a terminal only when that shows none.
Bound, stated in the flag help and the verb's `Long` text: the flag only chooses where the same `lyx batten run` executes and never changes the run; the window lives as long as the reed session.

`orch-template-role.md`'s start step replaces its `setsid nohup` instruction with `lyx batten run <slug> --window`, saying the window list then names every run in flight; the stencil's claims in `claims_test.go` gain a claim pinning `--window` and a `mustNot` on `setsid nohup`.
Update the batten verbs line of `docs/overview.md` and the run-start paragraph of `internal/battenshed/doc.go`.

Tests in `cli_test.go` with a fake opener: a fresh window prints the envelope naming it; an existing live window prints "nothing started"; the no-session, missing-capability, failed `new-window` and read-back refusals each print their way forward; the opener receives `batten:<slug>` and `batten run <slug>` without `--window`; the flag on any other verb is unknown.

**ImpactSummary:** the run verb of `lyx batten` gains one flag and a second body; the orch role stencil's start step changes; no other verb or caller changes.
