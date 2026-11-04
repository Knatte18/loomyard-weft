# Prime `board` button opens a short-lived board agent

Board work, such as filing, merging, promoting or releasing a few entries, went through the orch.
The orch is a long-lived session that carries every run's notices and rules, which is heavy for a talk about one or a few notes.
The prime's status bar now has a `board` button that opens an ordinary, short-lived agent for that work.
The board holds the agent's state, so it needs no handoff and no context cycle.

## What changes

- **`board` button.**
  `internal/reedengine/template_posix.yaml` and `template_windows.yaml` give the `prime` group buttons in place of the empty list: `code`, `terminal`, `agent`, `board`, `distill` and `refresh`.
  `board` uses the `ask` prompt (`Board agent name:`, default `board`) and runs `lym shuttle open --role {answer} --model 'opus[high]' --skill ly:board-policy --skill ly:board`.
  No new role: the skills make it a board agent, so the same way can make other kinds of agent.
- **`lym shuttle open --skill`.**
  The flag is repeatable, and the skills load in the order given, in the opened session's one first turn.
  `shuttleengine.OpenSpec` gains `Skills`.
  `claudeengine` puts the skill-load message on the fresh launch line as the first turn and on no resume line.
  `Runner.Open` refuses a non-empty list for an engine that cannot load skills, and the engine refuses a message past the launch limit, both before anything is created.
- **`ly:board-policy` skill.**
  The policy for keeping the board moves out of the orch role's "The board" section and the orch's rule list into `plugins/ly/skills/board-policy/SKILL.md`, listed in `INDEX.md`.
  `ly:board` stays the mechanics only, since run agents load it to file notes.
  The orch role now points to the skill, keeping only its own rules: recording a decision on a run's task, and no exploration as a background agent.
  `orchSkills` in `internal/orchcli/start.go` loads `ly:board-policy` into the orch session.
- **Board agent's own section.**
  The skill's last section covers a session opened for board work: read the board and code itself, talk each case through before writing, split an entry into tasks on approval, and hand a case about a run in flight back to the operator naming the orch.
- `docs/overview.md` and the `orchcli`, `shuttleengine` and `claudeengine` package docs describe the new flag and skill.

## Design decisions

- The policy lives in one skill that both the orch and the board agent load, so it cannot drift between two copies.
- The orch keeps decisions on a task a run holds and anything about a run in flight; the skill says so, and the board agent hands such cases back.
- Only the prime carries the `board` button: a board agent opened in a beam carries the beam's slug, and the board refuses a task write from a slug-carrying session.
- The first-turn skill load is the only turn an opened session takes, so `lym shuttle open` stays a no-prompt open for callers that pass no `--skill`.

## To check by hand

- Click the `board` button in a live prime: it should ask for a name, open a window, and the agent should have both skills loaded before its first reply.
- Confirm the orch session still gets `ly:board-policy` after a restart and behaves as before on release and bundling.
- The orch role lost two lines the skill now carries, `Never set a priority on a board entry unless the operator names one` and `never set recipe: darn on a board entry yourself`; check that the skill's wording matches the intent and that the rule stays in force for an orch that loads the skill.
- `template_windows.yaml` carries the same buttons and the same `cd {worktree} && …` `run` lines as the posix template; no Windows session ran them.
