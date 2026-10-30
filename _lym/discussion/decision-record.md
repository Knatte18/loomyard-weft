# Decision record — notices-from-dobby

## Goal

Dobby detects and sends the quiet, driver-dead, api-error and usage-limit notices of a running child in its own loop over the runs it drives, as `[dobby notice]` lines from `<shortname>:DOBBY`, and batten's Run-Shed row no longer detects them.

## Scope

- In: the running child's conditions in `internal/battenshed/notice.go` (quiet, driver-dead of a running child, api-error with its revive streak, usage-limit), moved into `internal/dobbyengine` with their keys, precedence and idle clocks unchanged.
- In: the seams dobby needs to read a run's child, filled in `internal/beamcli/dobbydeps.go` over the reads batten's wiring in `internal/battencli/wire.go` already makes.
- In: the sent-notice marker moved to the run's durable records directory under `_lym`, committed with batten's status.
- In: the orch role's Notices and stop sections in `contracts/stencils/orch/orch-template-role.md`, the orchcli doc, the battenshed and dobbyengine package docs, and the recipe comments that name the moved notices.
- Out: the state-changed notice and its driver-not-revived follow-up, the busy-pane notice, the owner-elsewhere notice and the Land, Worktree-Teardown and hand-back stop notices; they stay batten's, under `[batten notice]`.
- Out: the hub's deployed stencil copies, which the landing's deploy rewrites from `contracts/`.

## Decisions

### D1 — Dobby owns the running child's four conditions; batten keeps the notices of its own steps

Dobby sends quiet, driver-dead of a running child, api-error and usage-limit, and the api-error revive that precedes an api-error notice.
Batten's notice step keeps only the notices that follow a step the pass itself takes or holds: the state-changed notice with its stop-report, arbiter and relay holds, the driver-not-revived follow-up of a halted child whose revive the pass made, the busy-pane notice of a refused resume, the owner-elsewhere notice and the row and hand-back stop notices.
Rationale: the four conditions need a continuous watch over a run whose state does not change, which is the daemon's job; the others are each tied to a pass's own action and are seen by the pass that takes it.

### D2 — The detection runs in dobby's tick, on probe ticks, over every running run it drives

The detection lives in `internal/dobbyengine` as a new file, and each probe tick runs it for every listed run whose batten state is running with no cancel marker.
A run whose child status is absent or not running, or that a pause covers, is in no condition, as in batten.
The checks, their order (dead driver first, then usage-limit, api-error and quiet), the episode keys, the quiet window and the api-error idle time, and the idle clocks that subtract recorded sleeps and paused intervals, move unchanged.
The readings come through new Deps seams that read one run's child by slug, revive its sessions stalled on an API error, and give the quiet window; dobby's own recorded sleeps feed the idle clocks.
dobbyengine declares its own reading types, since its import allowlist admits neither battenshed nor shedengine.
The quiet window stays the batten recipe's Run-Shed `notice_quiet_min`, read by the hub wiring as it already reads `notice_probe_s`.
The driver-dead text names the respawn cap when batten's memory record shows the cap reached, read through an exported battenshed reader.
Bound: the detection adds per probe tick one child read per running run, the same reads a pass made per probe; it sends nothing and changes no run state beyond the api-error revive batten already sent.

### D3 — The sent-notice marker lives in the run's records directory under `_lym`

The marker, one episode key per line, is a file in the run's durable directory in the prime, its path declared in `internal/shedrun` beside the status and seed.
Batten's status commit carries it beside the status and seed whenever it exists, so it reaches git and the next machine's dobby, and `lym batten cleanup` removes it with the run directory.
Dobby's episode memory, the child state it last saw, when that episode began and the api-error revive streak, rides in the run's entry of dobby's own state record, so a re-exec keeps it.
Each notice's delivery attempts are kept in process.
Bound: between a marker write and the next batten transition the marker is an uncommitted change in the prime, as the status file already is while a run is watched; a machine switch in that gap can repeat one notice.

### D4 — The lines read `[dobby notice]` and go through dobby's Notice seam

A moved notice reads `[dobby notice] <slug>: <condition>; …` with the same body batten rendered, and goes through dobby's existing Notice seam, from `<shortname>:DOBBY` or `<shortname>:<beam>:DOBBY`, to the nearest live orch of dobby's worktree.
A notice that is not posted is tried again on the next probe tick, at most three times, then dropped with a Warn, as batten dropped one.
Batten's remaining notices keep `[batten notice]`.

### D5 — The orch's rules and the docs name both lines

The orch role's Notices section lists the dobby notices under dobby's own lines and the remaining batten notices under `[batten notice]`; its stop section counts a `[dobby notice]` as a stop.
The battenshed and dobbyengine package docs, the orchcli doc and the batten recipe's comments on `notice_quiet_min` follow the move.

## Constraints

- `PATTERN-cwd-resolution`: dobbyengine takes every path from its caller and imports no CLI package or `internal/lymcwd`; the hub wiring in `internal/beamcli` resolves the paths.
- `PATTERN-durable-vs-ephemeral-state`: only `internal/lymdirs` names `_lym`; the marker's path is built in `internal/shedrun` from its existing run-directory accessor.
- `PATTERN-tolerant-records`: the new state-record fields are added to dobby's tolerant record through `internal/state`.
- `PATTERN-agent-messaging`: the notices go as creel mail through dobby's Notice seam.
- `PATTERN-test-speed`: the moved behaviour's tests are tier0, with injected clocks and fake readings, and run through `lym gate test`.

## Auto-mode assumptions

- "Quiet, driver-dead, api-error and usage-limit" names the running child's conditions; the halted child's driver-not-revived notice is part of the state-change chain the pass owns.
- "Lives under `_lym`" means a tracked path that reaches git, which batten's status commit gives the marker.
- The quiet window keeps its config key on the Run-Shed row, since dobby already reads its probe interval there.

## Open risks

- The prime carries the marker as an uncommitted file until the next batten transition commit.
- Dobby's first sight of a running child after a machine switch dates its episode from that sight, so a driver-dead key the other machine sent can be sent once more.

## Acceptance criteria

- D1: batten's Run-Shed row sends no quiet, api-error, usage-limit or running-child driver-dead notice and makes no api-error revive; its state-changed, driver-not-revived, busy-pane and owner notices behave as before.
- D2: a dobby tick test covers each moved condition with a fake child reading: dead driver, the respawn-cap text, usage-limit over api-error over quiet, quiet with no runs read, a live wait marker that holds quiet, a paused child, and sleeps and paused intervals that delay a notice.
- D2: the api-error revive revives on the first sight and notices on the next sight of the same episode, and on the third revived episode in a row with no child status write between.
- D3: a sent notice's key is written to the marker path the wiring gives, a second tick in the same episode sends nothing, a new episode drops the old key, and batten's status commit paths include the marker when it exists.
- D4: a moved notice reads `[dobby notice] <slug>: …`, goes through the Notice seam, is retried on later probe ticks and is dropped after three failed attempts.
- D5: no rule, stencil or doc in the repository names the moved notices as `[batten notice]` lines.
