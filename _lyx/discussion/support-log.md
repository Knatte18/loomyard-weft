# Support log — agent-naming

## Interview

The session ran autonomously, so every turn below is a question the Discussion agent put to itself after reading the board entry and the code, with its own pick.

1. **Is this the right thing to build?**
   The board entry records operator decisions from 2026-10-01/02 that fix the grammar, the short code and the delivery channel, and it names this task as the prerequisite for orch-discussion-review's live question mode.
   Pick: build it as decided; the open points are the ones the operator left to Discussion.
2. **Where does the short code live, and who reads it?**
   `.lyx-warp` sits at the board root on weft:main, owned by `internal/fabricengine`; `.lyx-anchor` is read by `internal/lyxcwd` because resolution needs it.
   The code is not needed for resolution.
   Pick: `.lyx-code` beside them, owned by fabricengine; `hubgeom` reads it to tell reed.
3. **What does clone do in each binding state?**
   Pick: fresh bind requires `--code`; a recorded code is derived and a disagreeing flag refuses; a bound but uncoded weft takes the flag or warns; `--reset` keeps it.
4. **How do existing hubs get a code?**
   Pick: a new `lyx fabric code [<code>]` verb that prints or records, idempotent on equal input and refusing a change.
5. **What happens on a hub with no code?**
   The operator said no name can be formed without one.
   Pick: refuse the spawn with `lyx fabric code` as the remedy.
6. **Machine-wide short-code check?**
   No machine-wide hub registry exists, and two clones of one repo share a code by derivation.
   Pick: no check; recorded as an open risk.
7. **`-N` or a free suffix for several of one role?**
   Parallel webster workers and bouncer judges spawn several strands of one role in a worktree without the caller choosing names.
   Pick: both — reed auto-numbers from `-2`, and an explicit suffixed role is accepted but never renumbered.
8. **Fixed role vocabulary or open set?**
   The code already uses many role tokens (`discussion`, `plan`, `judge`, `triage`, `targeting`, `recovery`, `master`, `describe`, `conflict`, `burler`, `friction`, `rework`, `driver`, `orch`).
   Pick: open set under a grammar, with constants only for `orch` and `driver`.
9. **What happens to the round and the guid prefix in today's names?**
   Today's `strand_name` template is `<ROLE>:<ROUND>:<SHORT_GUID>`.
   Pick: retire the key and its tokens; names become addressable, `-N` handles concurrency.
10. **Who forms the name, given shuttle builds the launch line before reed adds the strand?**
    Pick: reed forms it in `AddStrand` under its lock; the Claude line references `$LYX_STRAND_NAME`, which reed injects, so the order problem disappears.
11. **Does `claude --name` exist, and where does Claude record the name?**
    Checked: Claude Code 2.1.287 has `-n, --name <name>`; transcripts record `{"type":"custom-title","customTitle":...}` entries (seen in an existing `quarry:orch` session).
    Pick: launch and resume lines carry `--name`; drift detection reads the latest `custom-title`.
12. **How does a provider-blind reed restore a Claude session name?**
    Detection, idle check and the `/rename` command are all Claude-specific, and shuttle imports reed, not the reverse.
    Pick: reedengine declares a seam, cliwire fills it with the Claude engine's implementation, and the watchdog tick drives it.
13. **Where is the parent recorded, and how does reed learn it?**
    The board says the run seed; `shedrun.Seed` is write-once with `recipe`, `driver` and `params`.
    Pick: an optional `parent` field filled from `$LYX_STRAND_NAME` at first seeding; batten copies its parent into the child seed; the worktree's parent is its `self` run's; `hubgeom.ReedGeometry` tells reed.
14. **Does a resumed start rewrite or refuse on a different parent?**
    Pick: neither; the first seeding wins, so an operator resuming a hub-spawned run keeps the hub as parent.
15. **How does the hub get its name?**
    A hand-started Claude session cannot export env to its Bash calls, since shell state does not persist between them.
    Pick: `lyx orch start`; reed names the orch strand `<code>:orch`.
16. **Which stencils send to the parent?**
    Producer stencils are autonomous file-writers; the asker in a worktree is the driver.
    Pick: the ly-drive skill and the loom driver prompt send a short notice on escalation, keeping the stop report as the fallback.
17. **Is `lyx reed list` per worktree or hub-wide?**
    No list verb exists today; the hub needs to find strands in other worktrees.
    Pick: hub-wide.
18. **Testing approach.**
    Pick: unit tests for agentname and the Claude provider; integration tests against real git for fabric and against real tmux for reed; a fake provider for the watchdog seam; help-tree tests for the new verbs.

## Rejected alternatives

- **Fall back to the repo name when no code is recorded.**
  Rejected: the operator decided no name can be formed without the code, and a fallback would mint names that change once the code is set.
- **Read `.lyx-code` in `internal/lyxcwd` next to the anchor.**
  Rejected: the Cwd Resolution Invariant keeps lyxcwd to resolution, and resolution does not need the code.
- **A machine-wide code registry with a refusal on clash.**
  Rejected: no registry exists, and the common clash, two clones of one repo, is legitimate and shares the code by derivation.
- **Only `-N`, or only free suffixes.**
  Rejected: only-suffix forces every parallel caller to invent names; only-`-N` leaves no way to give a second long-lived role a meaningful name.
- **A closed role vocabulary in agentname.**
  Rejected: every new producer would need a registry edit, and only `orch` and `driver` are addressed by convention.
- **Keep the round and guid prefix in the name.**
  Rejected: names would be unguessable and different on every round, which defeats addressing.
- **The caller forms the full name before `Prepare` and passes it to reed.**
  Rejected: `-N` assignment would race between concurrent spawns in one worktree outside reed's lock.
- **reedengine detects and restores Claude session names itself.**
  Rejected: reed is a provider-blind carrier, and the detection reads a Claude transcript.
- **A Claude Code hook in the engine's settings file restores the name after compact.**
  Rejected: no hook output that sets a session name could be confirmed, and the operator assigned the repair to the watchdog.
- **Pass every strand's parent through `AddSpec`.**
  Rejected: every spawn site would need to read the seed; a worktree-level parent told through geometry is read once.
- **A general resolver for the conversational short form.**
  Rejected: two-segment short forms (`landing:driver`) collide with two-segment full prime names (`mh:orch`), and lyx writes full names wherever something is addressed, so no lyx code needs the resolver.
- **A `--parent` flag on `batten run` and `loom start` for hand-started hub sessions.**
  Rejected under YAGNI: `lyx orch start` is the supported hub, and its migration path from a hand-started session exists.
- **An identity notice appended to every Claude strand's system prompt.**
  Rejected: autonomous producers would start sending notices to the hub, which is noise until orch-discussion-review designs the live question mode.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 Short-code grammar.** Resolved, auto-pick: `[a-z][a-z0-9]{1,5}`; short, separator-free, fits the operator's examples.
- **Q2 Record location and owner.** Resolved, auto-pick: `.lyx-code` at the board root on weft:main, owned by fabricengine.
- **Q3 Clone behaviour per binding state.** Resolved, auto-pick: as in the decision record's Short code section.
- **Q4 Existing hubs.** Resolved, auto-pick: `lyx fabric code [<code>]`.
- **Q5 Spawn on an uncoded hub.** Resolved, auto-pick: refuse with remedy; follows the operator's "no name without the code".
- **Q6 Machine-wide short-code check.** Resolved, auto-pick: none; left to Discussion by the operator.
- **Q7 `-N` or free suffix.** Resolved, auto-pick: both; left to Discussion by the operator.
- **Q8 Role vocabulary.** Resolved, auto-pick: open set under a grammar, constants for `orch` and `driver`.
- **Q9 Round and guid in names.** Resolved, auto-pick: retire with the `strand_name` key.
- **Q10 Where the name is formed.** Resolved, auto-pick: reed's `AddStrand`, with the Claude line referencing the injected env.
- **Q11 Session-name repair owner.** Resolved, auto-pick: watchdog through a reedengine seam filled by cliwire.
- **Q12 Parent record and propagation.** Resolved, auto-pick: optional seed `parent`, first seeding wins, batten copies into the child, worktree parent from `self`.
- **Q13 Hub name source.** Resolved, auto-pick: `lyx orch start`.
- **Q14 Which stencils send to the parent.** Resolved, auto-pick: ly-drive skill and loom driver prompt only.
- **Q15 `lyx reed list` scope.** Resolved, auto-pick: hub-wide.
- **Q16 tmux `allow-set-title` availability on the Windows tmux build.** Open: carried to Open risks; the plan should check the tmux version reed requires.
- **Q17 Whether `/clear` resets the Claude session name.** Open: unverified; the watchdog repair covers both outcomes.
