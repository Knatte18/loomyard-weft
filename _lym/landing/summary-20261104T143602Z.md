# Prime status-bar button opens a board agent; board policy gets a skill

Talking through one or a few board entries no longer needs the orch.
The prime's reed status bar gains a `board` button.
It opens an ordinary agent, the same kind `lym shuttle open` opens, with two skills loaded: the new `ly:board-policy` and `ly:board`.
The orch now loads `ly:board-policy` too, so the board policy is written in one place.

## What changes

- **`lym shuttle open --skill <name>`**, repeatable.
  The skills go on `OpenSpec.Skills`.
  `Runner.Open` refuses them before it creates anything when the engine lacks `SkillLoader`.
  Claude's `OpenLaunch` puts the skill-load message on the fresh launch line as the session's first turn, the same way a run's launch does, and refuses a message over the launch limit.
  The resume line carries no message, so a resumed session does not load the skills a second time.
- **New skill `plugins/ly/skills/board-policy/SKILL.md`.**
  It holds the policy that was in the orch role's "The board" section: one finding per note, a body holds the current design only, propose bundles and wait for approval, and release only on a named go-ahead.
  It also takes the priority, recipe, profile and drift-scope rules.
  A closing section tells a session opened from the button what to do.
  It reads the board and code itself, talks each case through before writing, and splits an entry into tasks on the operator's approval.
  It sends cases that belong to the orch back to the operator, and it ends without a handoff.
  `ly:board` is unchanged and holds only the board mechanics, so run agents that load it to file notes do not get the policy.
- **Orch role** (`contracts/orchrole/orch-role.md`).
  "The board" now points to `ly:board-policy` and keeps only the orch's own rules: recording decisions on held tasks or beams, and never running an exploration as its own background agent.
  The priority rule and the darn-recipe rule moved into the skill.
  `orchSkills` adds `ly:board-policy`, so the orch loads it at every launch and at every skill reload.
- **reed templates** (POSIX and Windows).
  The `prime` group is no longer empty.
  It lists the beam set (code, terminal, agent, distill, refresh) plus `board`, which asks `Board agent name:` (default `board`) and runs `lym shuttle open --role {answer} --model 'opus[high]' --skill ly:board-policy --skill ly:board`.
- `docs/overview.md`, the `orchcli` package doc, the shuttle and claudeengine package docs, the plugin skill index and the `open` help text describe the above.

## Design decisions

- **Only the prime gets the button.**
  A session opened in a beam carries the beam's slug in its strand name.
  The board's task guard treats any slug-carrying session other than an orch as a run agent and refuses its task creation and promotes.
  So a board agent in a beam could not promote notes.
  In the prime the name is `<shortname>:<role>` with no slug, so promotes and releases work.
- **The skill load happens at launch.**
  It is not typed into the pane after start.
  This matches how `Runner.start` delivers a run's skills (PATTERN-parent-directive: skills go on the launch spec), and it needs no wait for the pane to be ready.
- **Split parts are not flagged discussed.**
  Only lym writes the `discussed` flag and the stored record.
  For a discussed entry, the skill tells the agent to put each part's share of the Sketch in that part's body, and each part then runs its own Discussion.
  The note `discussed-task-split` describes a split into discussed parts, which still needs lym to write the record for each part.

## Open risks

- **Codex.**
  The peer task `codex-trial-1-fixes` gives Codex an `OpenLaunch`.
  Once both land, that `OpenLaunch` must honour `spec.Skills`, or a Codex open with `--skill` will ignore them without any error.
  `Runner.Open` only checks that the engine is a `SkillLoader`, and Codex is one.
- **Hubs with a custom beam group.**
  A hub layer that customised the `beam` buttons but set no `prime` group used to see its own beam set in the prime.
  It now sees the shipped prime set, because a group written in a layer replaces only that group.
  Such a hub can copy its beam entries into a `prime` group.
