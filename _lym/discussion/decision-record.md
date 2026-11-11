# Decision record — claude-account-weights

## Goal

Spread Claude usage over several Claude accounts on one machine.
Each worktree runs every Claude session it starts under one account.
That account is picked at the worktree's first fresh session, by a configured weight and the account's current headroom in each of its limit windows.

## Scope

In:

- A list of accounts in the `claude` config module's machine layer: name, config directory, weight and limit windows.
- The per-worktree pick, its persisted record, and the hub-wide weighted counter it reads and advances.
- A headroom source: Claude Code's status line input, recorded per account.
- Setting the account's config directory in the claude engine's launch, so every session of the worktree runs under it.
- Every lym reader of Claude's per-config-directory files reading the right directory: transcripts, the session registry, resume, the installed-plugins lookup, `lym report` and `tools/tokencount`.
- The rule for the prime's and each beam's long-lived sessions: orch and dobby.
- Tests against fake config directories and a fake status line input, and a manual check on the operator's second account.

Out:

- Logging the accounts in, preparing a new config directory, and whether the use is within the accounts' and the organisation's terms: the operator's to settle.
- Moving a worktree to another account when its account hits a limit.
- Any engine other than Claude.
- Standalone runs, outside a hub: they have no machine layer, load no accounts and keep today's behaviour.

## Decisions

### D1 — The accounts are a machine-layer list in the `claude` config module

`claude.accounts` is a list.
Each entry holds `name`, `config_dir`, `weight` and `windows`.

- `name` matches `[a-z0-9-]+` and is unique in the list.
- `config_dir` is an absolute path; a leading `~/` expands to the user's home directory.
  Every comparison of config directories, in this record and the code, is between paths that are expanded, made absolute and cleaned.
  Two entries never name the same directory.
- `weight` is a positive integer: the account's long-run share is its weight over the sum of the weights.
- `windows` maps a window name to the used percentage at or above which the account is passed over in that window.
  The window names are a closed set, `five_hour` and `seven_day`, the two Claude Code's status line reports.
  Each value is an integer from 1 to 100.
  An account is judged only by the windows it lists, so an account without `five_hour` is never judged by one.
  There is no default threshold.

The key is classified `CategoryMachine`, so it is read from `<Bolt>/.lym/config/claude.machine.yaml`, which is never tracked.
The template default is `[]`.
A non-empty value from any tracked layer, the worktree's or the hub's `claude.yaml`, is refused when the module loads, told by the layer provenance `configengine` reports for each unit.
The refusal names the key and the way forward, `lym config claude --layer machine`.
An invalid list, such as a duplicate name, a relative path or an unknown window, is refused the same way and fails the session launch; it never degrades to no accounts.

The machine layer is per hub, so each hub on a machine lists its accounts in its own file.

Rationale: the accounts and their directories are facts of the machine, and account details never go into a file pushed to GitHub (board entry, operator 2026-11-07).
The machine layer is the one never-tracked layer `PATTERN-config-layers` opens to such a key.
A window threshold per account names which windows an account has and how close to a limit it may run in one value.

### D2 — The account is recorded once per worktree and kept while its directory exists

A worktree's account is recorded in a never-tracked file under its anchor, `.lym/claude/account.json`, named through `internal/lymdirs` and written through `internal/state`.
It holds the account name, empty when no listed account names the directory, the config directory and the time it was recorded.
Every Claude session of the worktree runs under the recorded directory: writer, reviewers, fixers, the loom driver, the orch and the dobby.

How a session launch reads it:

- A record whose directory exists is kept, whether or not `claude.accounts` still lists its account.
  Removing an account from the list, or emptying the list, stops new picks of it and leaves the worktrees already on it where they are.
- A fresh launch with no record, or with a record whose directory no longer exists, picks an account (D3) and writes the record.
  A record replaced this way is logged.
- A resume launch with no record makes no pick.
  It records the directory that holds the session's transcript, searched among the listed accounts' directories and the default directory (D6).
  A session started before this change is found in the default directory.
  A transcript found nowhere is recorded as the default directory, and the resume check refuses as it does today.
- A resume launch whose recorded directory no longer exists is refused, naming the directory and the way forward: restore the directory, or remove the record so the next fresh session picks again.

A resume, a re-attach, a reed relaunch and a hand pass read the record and keep the directory.

Rationale: all sessions of a run must see each other's transcripts and resume their own, and Claude Code keeps transcripts per config directory (D4).
A pick on a resume would move a session away from its own transcript, which the resume check refuses.
A per-worktree record dies with the worktree, so nothing accumulates for landed tasks.
The record is a machine-local cache, which `PATTERN-lym-files` allows under `.lym`: a run is started and resumed only on its owner machine (`PATTERN-owner-machine`), and that machine keeps the file.

### D3 — The pick is a smooth weighted counter that passes over an account short of headroom

A hub-wide, never-tracked file `<Bolt>/.lym/claude/account-picks.json` holds one pick count per account name.
A pick runs under one `internal/lock` write lock on that file, and does these steps:

1. It reads the worktree's record again under the lock; a record a concurrent session wrote meanwhile, whose directory exists, wins and no pick is made.
2. An account is a candidate when it is logged in.
   On Linux and Windows that is when its `config_dir` holds Claude Code's credentials file, `.credentials.json`, which lym stats and never reads.
   On macOS Claude Code keeps the login in the Keychain, so every listed account is a candidate.
   The stat proves the file exists, not that the login is current; an expired login meets Claude Code's own login prompt in the pane.
   With no candidate, the session launch fails, naming the accounts and their directories.
3. A candidate is eligible when every window it lists has headroom (D5).
   Among the eligible candidates it takes the one with the lowest count over weight, ties going to the earlier entry in the list.
4. When no candidate is eligible, it takes the candidate whose blocking windows all reset soonest, and logs that every account was short.
5. It increments the taken account's count and writes the worktree's record, then releases the lock.

A name the counter file lacks starts at its weight times the lowest count over weight among the listed accounts that have a count, so a newly added account takes its share from the next pick and does not catch up on past picks.
Counts for names no longer listed are kept and ignored.

The counter and the headroom records (D5) are per hub, like the machine layer: the share holds within a hub, and a hub learns of another hub's use of an account only through the figures its own sessions see.

Rationale: a counter keeps the long-run share at the weights, which a hash of the slug cannot do once accounts are skipped.
Count over weight spreads the picks evenly, for example A, B, A for weights 2 and 1, rather than in bursts.
Falling back to the soonest reset keeps work starting when every account is near a limit (D8).

### D4 — The claude engine's launch sets the account's config directory

The claude engine resolves the worktree's directory (D2, D3) when it prepares a session.
It puts the export of `CLAUDE_CONFIG_DIR` into the launch command and the resume command it builds.
No caller, and neither `internal/reedengine` nor `internal/hubgeom`, names the variable.

- The export comes after the `env` module's exports in the pane's launch line, so the recorded directory wins over an `env` module value of `CLAUDE_CONFIG_DIR`.
  With no record and no accounts, the engine exports nothing and an `env` module value stands as today.
- A recorded directory equal to Claude Code's default, `~/.claude`, gets `CLAUDE_CONFIG_DIR` unset in the launch rather than set.
  Setting it to the default would move `.claude.json` from `~/.claude.json` into `~/.claude/`, which loses that account's existing state.
- The variable sits in the launch command itself, so a reed relaunch that replays the stored command keeps the directory.

Facts this rests on (Claude Code 2.1.322 installed; docs at code.claude.com, read 2026-11-11):

- `CLAUDE_CONFIG_DIR` selects a directory with its own settings, session history and claude.ai login (Authentication docs, "Log in with multiple accounts").
- On Linux the login is kept in `<config dir>/.credentials.json`; on macOS in the Keychain, keyed to the directory (same page, "Credential management").
- Every `~/.claude` path moves under it, transcripts in `projects/` and the session registry included, and so does `.claude.json`, which otherwise sits at `~/.claude.json` (claude-directory docs).

Rationale: the board asks for one place in the claude engine's launch, and `PATTERN-shuttle-provider-seam` keeps provider specifics such as this variable inside `internal/shuttleengine/claudeengine`.

### D5 — Headroom is read from Claude Code's status line input, recorded per account

When the worktree's record names a listed account, the session settings file the claude engine renders and passes with `--settings` carries a `statusLine` command.
The command runs a lym verb with that account's name as its argument.
The verb reads the status line JSON on stdin and takes `rate_limits.five_hour` and `rate_limits.seven_day`, each `used_percentage` and `resets_at`.
It merges them into `<Bolt>/.lym/claude/usage/<account>.json` under an `internal/lock` write lock, through `internal/state`, window by window:

- an incoming window with a later `resets_at` than the stored one replaces it;
- one with the same `resets_at` replaces it only when its `used_percentage` is higher;
- one with an earlier `resets_at` is dropped.

So an idle session that repeats older figures never lowers what a busier session recorded.
It prints one short line: the account name and each window's used percentage.
An input without `rate_limits` writes nothing and prints the line without figures.

A window has headroom when its record is absent, when its `resets_at` has passed, or when its `used_percentage` is below the account's threshold for it.
So an account no session has used yet, or one idle past its reset, is eligible.

In lym's agent panes this status line replaces the operator's own, since settings passed with `--settings` take precedence over user settings, and it does not chain to the operator's command.

Facts this rests on: the status line input carries `rate_limits.five_hour` and `rate_limits.seven_day` with `used_percentage` (0 to 100) and `resets_at` (Unix seconds) since Claude Code 2.1.80.
It carries them only for subscription accounts and only after the session's first API response.
Either window may be absent on its own, and one past its `resets_at` is dropped (statusline docs; the installed 2.1.322's changelog).
No documented route reads an account's usage without a running session; `/usage` is interactive only.

Rationale: the status line is the one documented source that gives the session and weekly figures the operator asks for (board entry, operator 2026-11-11).
The figures go stale while an account is idle, and the reset times bound that staleness: a passed reset reads as free.
Naming the account in the command, rather than matching a directory, means a path spelled differently can never leave an account unrecorded.

### D6 — Every lym reader of Claude's config directory reads the right directory

The readers that today take `$HOME/.claude` read the right config directory:

- A reader told the worktree, such as the transcript readers, the resume check and the installed-plugins lookup, takes the worktree's recorded directory.
- A reader told only a session id, such as the prober's liveness read, looks the id up in the session registry of every known directory; session ids are UUIDs, so one match is the session's.
- `lym report` and `tools/tokencount` read the `projects/` directory of every known directory, each once, and count a session id found in two of them once.

The known directories are the listed accounts' directories, the `env` module's `CLAUDE_CONFIG_DIR` when set, the process's own `CLAUDE_CONFIG_DIR` when set, and the default `~/.claude`.
A reader with no record to go by takes the `env` module's value, then the process's, then the default, as Claude Code itself would for a pane.
`tools/tokencount` run outside a hub has no machine layer and reads the process's `CLAUDE_CONFIG_DIR` and the default alone.

The hook-driven readers already take `transcript_path` from the hook payload and need no change.

Rationale: under a second config directory the transcripts, the session registry and the plugin list live there (D4), so a reader of `~/.claude` alone loses every session on that account: resume, liveness, usage records and cost reports.
The `env` module's value reaches only panes, gate tiers and Drift steps, never lym's own process, so the readers read it from the module.

### D7 — The prime's and each beam's long-lived sessions follow the same per-worktree rule

The prime worktree and each beam worktree record their directory like a task worktree (D2).
The orch, the dobby and any other session there run under it.
A beam's account is independent of the prime's.
Their sessions started before this change resume and record the default directory (D2), and they keep it until the operator removes the record.

Rationale: an orch and a dobby are resumed across restarts, and a resume needs the session's transcript, which only its own directory holds.
A switch on each fresh start would split one worktree's sessions over two directories, and the readers of D6 read one directory per worktree.

### D8 — A usage limit is reported and waited out, never moved to another account

A session that hits its account's limit is handled as today: the engine reports the limit and its reset, and the orch watcher waits it out (`internal/orchengine/usagelimit.go`).
Lym does not move a worktree or a session to another account.

Rationale: a session's history is in its account's config directory, so a session moved to another account cannot resume.
The pick's headroom check keeps new worktrees off an account near a limit, which covers the case the move would serve.

### D9 — Tests run on fake config directories; a manual check runs on the operator's second account

Automated tests use fake config directories under the test's temporary directory, each with an empty `.credentials.json` where the test needs a logged-in account.
They feed the status line verb fake JSON on stdin.
No automated test reads a real account or calls Claude.

The second account is wired for the manual check this way, by the operator, once before landing:
the operator logs it in with `CLAUDE_CONFIG_DIR=<dir> claude` and `/login`;
links `<dir>/plugins` to `~/.claude/plugins`, so lym's deploy, which syncs plugins into `~/.claude` alone, updates both accounts;
links the user-level instruction file and settings from `~/.claude` as wanted;
and lists both accounts with `lym config claude --layer machine`.
The check then starts two task runs and confirms each worktree's record names an account, a session's transcript lands in that account's `projects/`, and the hub's usage directory gains a file per used account.

Rationale: the board says the build is tested against fake directories and a fake usage source unless a real second account is wired, and that the task states how it is wired.

## Constraints

- `PATTERN-shuttle-provider-seam`: `CLAUDE_CONFIG_DIR`, the credentials file name, the status line verb's input shape and the pick live in `internal/shuttleengine/claudeengine`.
  A verb wired outside it reaches it only through `internal/shuttleengines`.
- `PATTERN-config-layers`: the new key is classified in the module's `ConfigSpec` in the same change, as `CategoryMachine`; it is a new key, so no reinterpretation is needed.
- `PATTERN-lym-files`: every new path, the worktree's record, the counter file and the usage directory, is named in `internal/lymdirs`; lock paths come from `internal/lock`.
- `PATTERN-tolerant-records`: the record, the counter file and the usage files are JSON read tolerantly and rewritten through `internal/state`.
- `PATTERN-cli-cobra`: the status line verb is a cobra command under the one root.

## Auto-mode assumptions

- A worktree's account is sticky while its directory exists, the prime's and beams' included (D2, D7); the operator changes it only by removing the record file.
- The status line does not chain to the operator's own command (D5).

## Open risks

- The headroom view and the share are per hub (D3), and this machine runs several hubs; a hub that rarely uses an account can pick it while another hub has spent it, until its own session sees the figures.
- A report that the five-hour window was removed for some Ultra accounts in October 2026 is single-sourced and unverified; D1's per-account window list covers either answer.
- Claude Code may change the status line input's field names; the verb then writes nothing and every account reads as having headroom, so the pick falls back to the weights alone.

## Acceptance criteria

- D1: a `claude.accounts` list in the machine layer loads with each entry's name, directory, weight and windows; a duplicate name, a relative path, two entries naming one directory after cleaning, a non-positive weight, an unknown window or a threshold outside 1 to 100 is refused.
- D1: a non-empty `claude.accounts` from a tracked `claude.yaml` is refused, and the refusal names `lym config claude --layer machine`.
- D1: with `claude.accounts` empty and no record, a session's launch command neither sets nor unsets `CLAUDE_CONFIG_DIR`.
- D2: a second session started in a worktree after the first uses the recorded directory, without a new pick.
- D2: a record whose account was removed from the list, its directory still present, is kept and its directory exported.
- D2: a fresh launch with a record whose directory is gone picks again and replaces the record; a resume launch with such a record is refused, naming the directory.
- D2: a resume launch in a worktree with no record, its transcript in the default directory, records the default directory and advances no count.
- D3: with weights 2 and 1 and both accounts eligible, six picks give the first account four and the second two, never two of the second in a row.
- D3: an account with a window at or above its threshold, its reset in the future, is passed over; the same account with its reset passed is eligible.
- D3: an account whose directory has no `.credentials.json` is never picked, by the weights or the fallback; with none having one, the launch fails naming the directories.
- D3: with every candidate short, the pick takes the candidate whose blocking windows reset soonest.
- D3: an account added to a list with existing counts is seeded at the lowest count over weight among the listed accounts that have one, never at zero.
- D3: two sessions starting at once in one worktree with no record end with one record and one counter increment.
- D4: a session in a worktree whose recorded directory is not the default gets a launch command and a resume command that export `CLAUDE_CONFIG_DIR` as that directory, after any `env` module value.
- D4: a worktree whose recorded directory is `~/.claude` gets a launch command that unsets `CLAUDE_CONFIG_DIR`.
- D5: the status line verb, given an account name and input with both windows, writes that account's usage file with both windows' percentage and reset; given input without `rate_limits`, it writes nothing.
- D5: a stored window is replaced by one with a later reset or a higher percentage at the same reset, and kept against one with a lower percentage at the same reset or an earlier reset.
- D5: a session whose worktree records a listed account gets session settings carrying the status line command with that account's name; with no listed account recorded, the settings carry none.
- D5: an account whose usage file lacks a window it lists, or has none, reads as having headroom in that window.
- D6: the engine's transcript reader, resume check and plugin lookup for a worktree recorded on a fake directory find the files in that directory and not under `~/.claude`.
- D6: the liveness read of a session id registered only under a fake account directory finds the session.
- D6: `lym report`'s cost reading counts the sessions under every known directory, the `env` module's included, a session id found twice counted once.
- D7: a prime or beam worktree's orch session and dobby session run under the worktree's recorded directory.
- D8: a session that hits a usage limit keeps its recorded `CLAUDE_CONFIG_DIR` across the wait and its resume.
- Review: D9's manual check on the real second account is written in the task's landing notes with its outcome.
