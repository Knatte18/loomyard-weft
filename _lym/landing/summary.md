# gatefixer: one cold fixer clears a failed writer gate

## What changes

When a writer's gate fails, a fresh, cheap agent now clears the mechanical failures before the writer hears of them.
Until now the writer fixed them in its own session at full context.
One shared package, `internal/gatefixer`, spawns or attaches to, waits on and stops the fixer strand and renders its prompt.
It serves four callers:

- the Darn-Write `verify` gate,
- the Plan-Write `plan` gate,
- the Discussion-Write `discussion` gate,
- the Webster card gate, which is the fork's own `lym gate test --gate card` call.

`lym webster fix-verify` moves onto the same core and keeps its verb, trigger, `verify-fixer` role, `recovery_timeout_min` and `VerifyFixers` records.
Its one behaviour change is that its strand is now registered with `internal/runproc` under the worktree's pair slug, so ending the pair's processes stops it.
`internal/websterengine/coldfixer.go` and `webster-prefix-verify-fix-cold.md` are removed; the cold frame now lives under `contracts/stencils/gatefix/`.

## Design decisions

- **The fixer never judges.**
  The caller re-runs its own gate after every fixer and acts on that result alone.
  What the fixer leaves goes to the writer with the fixer's reply, or a note that it ended dead.
- **Row gates block instead of returning pending.**
  A row gate element with `fixer` has its `Gate` and `Forked` closures wrapped (`gatefixer.WrapRowGate`, `internal/shedrecipe/entries_gatefixer.go`).
  On a failure at the writer's idle turn boundary the wrap spawns a fixer, waits, and re-runs the inner closure.
  Pending would re-run Darn-Write's `clean` gate over the fixer's uncommitted edits.
  `shuttleengine` is unchanged.
- **Only an idle-boundary evaluation spawns a fixer.**
  Deadline and restart finalizes run the inner closure through `Final` and only wait on a live fixer.
  Every other entry of the row waits on a live fixer before it runs, so each entry is judged over the finished tree.
  A restart while a fixer is live finalizes through the `Expired` probe.
  Fixers per row visit are capped at the entry's `attempts` plus one, because a failed re-prompt send re-evaluates without spending an attempt.
  A row run can outrun its deadline by up to one fixer timeout plus three runs of the inner gate.
- **The card gate hands off from the detached runner.**
  A red verdict of a capped `lym gate test` call on a key with `fixer` (`internal/gatecli/cardfixer.go`, `detachfixer.go`) spawns the fixer, then re-runs the same call.
  The front writes the caller's role into the run request.
  A strand whose role is in `gatefixer`'s fixer role set, numbered or not, never hands off.
  A call from the fork joins the run through the fixer phase whatever the fixer did to the dirty paths.
- **Config.**
  Each caller has its own model-spec key, defaulting to `sonnet[medium]`:
  - `darn.gate_fixer`,
  - `tapestry.plan_gate_fixer`,
  - `loom.discussion_gate_fixer`,
  - a gate.yaml `fixer` field, accepted on the `card` key only, which config load refuses elsewhere.

  An empty value turns that caller's fixer off.
  One `fixer_timeout_min` in gate.yaml, default 30, bounds the four callers.
  A caller with no pair slug spawns no fixer.
- **The fixer's limits rest on the prompt.**
  The body stencils set each caller's edit scope, commit rule and leave-list.
  The re-run judges only what the gate checks.
  The later review rows, or the fork's read of the uncommitted diff on the card gate, judge the fixer's edits.
  A forked Plan-Write evaluation that fails only on the missing lens review spawns no fixer.
- **Records.**
  Row fixer records live under the run's `.lym` scratch, keyed by row name, history length and the newest entry's `Repeats`.
  A goto ends every fixer role under the pair first (`internal/loomcli/fixerendguard.go`).
- **Writer stencils.**
  Plan-Write, Discussion-Write and the Discussion chair stop looping on their validator when the fixer is on and the worktree has a pair slug.
  The Webster fork stencil says the card gate has already been through a fixer.
  Text with the fixer off is unchanged.
- **Darn-Write's `clean` and `description` gates get no fixer.**
  Record-Write, Parts-Write, the review fix rows, Drift-Verify and Describe are untouched.

No issue is closed.

## To check by hand

- Run a Darn-Write, a Plan-Write and a Discussion-Write row with a failing gate and the fixer on, and confirm the fixer spawns, the re-run passes the row, and a still-failing re-run re-prompts the writer once with the fixer's reply.
- Run a Webster fork's red `lym gate test --gate card` call against a real hub.
  No `card` call is on record, so the 30-minute default timeout is unmeasured for that key.
- Kill the driver mid-fixer on a Darn-Write row, resume, and confirm it waits on the live fixer and does not halt on the `clean` entry.
- Findings left open:
  - `card9-slug-read-error-marked-transient`, in the pair-slug read of the card fixer path, which marks a slug read error as transient (BLOCKING design finding).
  - `card-gate-key-bare-literal`, a bare literal for the `card` gate key where a shared constant belongs (BLOCKING consistency finding).
  - `waitonly-marks-terminal-on-pass`, in the wait-only row wrap, which marks the fixer record terminal when its own closure passes (MEDIUM design finding).
- A pause that covers the run while a row fixer works can let the fixer reach its timeout.
  A writer whose prompt cache expires during a long fixer pays a cache write on its next turn when the fixer fails.
  A Sonnet fixer may clear fewer darn verify failures than the writer would.
- The verify gate failed once on `internal/configreg.TestModules_EveryTemplateKeyClassified/darn`.
  Commit `a84d2f2f` clears it by listing `darn.gate_fixer` among the hub-wide local-open keys; recheck that classification.
