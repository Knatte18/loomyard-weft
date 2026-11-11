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
- Every lym reader of Claude's per-config-directory files reading the right directory: transcripts, the session registry, resume, the installed-plugins lookup, the session-end usage records and `tools/tokencount`.
- The rule for the prime's and each beam's long-lived sessions: orch and dobby.
- Tests against fake config directories and a fake status line input, and a manual check on the operator's second account.

Out:

- Logging the accounts in, preparing a new config directory, and whether the use is within the accounts' and the organisation's terms: the operator's to settle.
- Moving a worktree to another account when its account hits a limit.
- Any engine other than Claude.
- Standalone runs, outside a hub: they have no machine layer, load no accounts and keep today's behaviour, but for reed's server no longer carrying a shell `CLAUDE_CONFIG_DIR` (D4).

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
A non-empty value from any tracked layer, the worktree's or the hub's `claude.yaml`, is refused when the module loads.
The value is read from each tracked layer's own document in the module's stacked layers, so a tracked value is refused even when the machine layer also holds the key.
The refusal names the key and the way forward, `lym config claude --layer machine`.
An invalid list, such as a duplicate name, a relative path or an unknown window, is refused the same way and fails the session launch; it never degrades to no accounts.

The machine layer is per hub, so each hub on a machine lists its accounts in its own file.

Rationale: the accounts and their directories are facts of the machine, and account details never go into a file pushed to GitHub (board entry, operator 2026-11-07).
The machine layer is the one never-tracked layer `PATTERN-config-layers` opens to such a key.
A window threshold per account names which windows an account has and how close to a limit it may run in one value.

### D2 — The account is recorded once per worktree and kept while its directory exists

A worktree's account is recorded in a never-tracked file under its anchor, `.lym/claude/account.json`, named through `internal/lymdirs` and written through `internal/state`.
It holds the account name, empty when no listed account names the directory, the config directory and the time it was recorded.
Every Claude session of the worktree runs under the recorded directory: writer, reviewers, fixers, the loom driver, the orch, the dobby and any session `lym shuttle open` opens there.

How a session launch reads it:

- With `claude.accounts` empty, as it always is outside a hub, a launch makes no pick, searches nothing and writes no record.
  A record already there whose directory exists is still read and its directory exported, so emptying the list leaves the worktrees already on an account where they are.
  A record whose directory no longer exists refuses the launch, a fresh one as well as a resume, as the gone-directory refusal below says, and nothing is exported or written;
  once the operator removes the record, launches export nothing.
  With no record, the launch exports nothing, and an `env` module `CLAUDE_CONFIG_DIR` stands as today.
- A record whose directory exists is kept, whether or not `claude.accounts` still lists its account.
  Removing an account from the list stops new picks of it.
  A record whose directory exists but is logged out is kept too: its sessions stop at Claude Code's login prompt,
  and the way forward is to log the directory in again or write the record with another `config_dir`.
  Removing the record moves only a worktree with no resumable session there, since the next fresh session's search finds the transcripts in that directory and records it again.
- With `claude.accounts` listing an account, a fresh launch with no record, or with a record whose directory no longer exists, first searches every known directory (D6) for the transcripts of the sessions this worktree's own lym state can resume:
  the ids every record of that state, wherever its owner keeps it for the worktree, stores for a provider session its owner later resumes, each read through its owner module's exported reader, the orch's state record among them.
  The shuttle run records and the reed strands alone fall short: shuttle sweeps a run's directory once its strand is gone, reed drops a strand once its work is done,
  and the sampler row's record and the PR-Review session record each keep a session id their owner resumes after that sweep.
  Plan's quarry lookups settle the full list of owners.
  The wiring that builds the run's engine set collects those ids through each owning module's exported reader and passes them to `Prepare` and `OpenLaunch` as told data in the launch spec,
  so neither `internal/shuttleengine` nor the claude engine reads any owner module's records.
  Transcripts an earlier worktree at the same path left behind are not searched for, so a task worktree recreated at a used path picks.
  When any directory holds them, it records the one holding the most recently written transcript and makes no pick.
  Only when none holds any does it pick an account (D3).
  A record replaced this way is logged.
- A resume launch with no record makes no pick.
  It records the directory that holds the session's transcript, searched among every known directory (D6), the `env` module's and the process's `CLAUDE_CONFIG_DIR` included.
  A transcript found nowhere writes no record: the resume runs under the directory D6's no-record fallback names, the `env` module's value, then the default,
  and the resume check refuses as it does today.
- A resume launch whose recorded directory no longer exists is refused, whatever `claude.accounts` holds, naming the directory and the way forward: restore the directory, or remove the record so the next fresh session searches and picks again, or, with the list empty, exports nothing.
  A resume after the removal writes no record of its own, since its transcript was in the gone directory, so the removal leaves the pick to the next fresh session.
- Shuttle runs the resume check before `Prepare`, so with `claude.accounts` listing an account the check itself resolves the directory by these rules before it stats the transcript:
  with no record it searches every known directory for the session's transcript, and with a recorded directory that is gone it makes the refusal above.
  `Prepare` then records the directory the same search finds.
  With `claude.accounts` empty the check stats the transcript under a record's directory when that directory exists, makes the refusal above when it is gone, and stats under the no-record fallback when there is no record; nothing is recorded.
  The check's live-holder test reads the session registries of every known directory (D6) as one set, by D6's combining rule, and refuses when any entry is live,
  whichever directory the check resolved and stats the transcript under.
  So a live holder under any known directory is caught, the default's included as today, also when one session id has transcripts in two directories.
  A registry directory or entry that exists but cannot be read warns and lets the check proceed, as today.

Every record write, a resume's and a fresh launch's alike, runs under the write lock D3 names and first reads the record again:
a record a concurrent launch wrote meanwhile, whose directory exists, wins and nothing is written.

A session started before this change has its transcripts in the directory it ran under, so the search records that directory whichever launch in the worktree comes first.

A resume, a re-attach and a hand pass read the record and keep the directory.
A reed relaunch replays the command its strand was launched with and reads no record (D4).
So a changed record, or a recorded directory that is gone, reaches a running strand only when a launch the engine builds starts it again, as a fresh session or a resume;
until then a relaunch keeps the old directory, and the refusal of a gone directory above does not run for it.
A relaunch over a gone directory can have Claude Code create it again, empty and logged out:
the strand stops at the login prompt, and the record, whose directory now exists, is kept.
The way forward for such a strand is to end it and start its session again with the lym verb that started it, after the operator logs the directory in or changes the record.

Rationale: all sessions of a run must see each other's transcripts and resume their own, and Claude Code keeps transcripts per config directory (D4).
A pick on a resume would move a session away from its own transcript, which the resume check refuses.
A per-worktree record dies with the worktree, so nothing accumulates for landed tasks.
The record is a machine-local cache, which `PATTERN-lym-files` allows under `.lym`: a run is started and resumed only on its owner machine (`PATTERN-owner-machine`), and that machine keeps the file.

### D3 — The pick is a smooth weighted counter that passes over an account short of headroom

A hub-wide, never-tracked file `<Bolt>/.lym/claude/account-picks.json` holds one pick count per account name.
A pick runs under one `internal/lock` write lock on that file, the lock every record write takes (D2), and does these steps:

1. Under the lock, it reads the worktree's record again and makes D2's search of the known directories for the transcripts of the worktree's resumable sessions, whose ids the caller told (D2).
   A record a concurrent launch wrote meanwhile, whose directory exists, wins; a directory the search finds is recorded; in either case no pick is made.
2. An account is a candidate when it is logged in.
   On Linux and Windows that is when its `config_dir` holds Claude Code's credentials file, `.credentials.json`, which lym stats and never reads.
   On macOS Claude Code keeps the login in the Keychain, so every listed account is a candidate;
   a listed account never logged in can be picked there, and its sessions stop at Claude Code's login prompt until the operator logs it in.
   The stat proves the file exists, not that the login is current; an expired login meets Claude Code's own login prompt in the pane.
   With no candidate, the session launch fails, naming the accounts and their directories.
3. A candidate is eligible when every window it lists has headroom (D5).
   Among the eligible candidates it takes the one with the lowest count over weight, ties going to the earlier entry in the list.
4. When no candidate is eligible, it takes the candidate whose latest `resets_at` among its blocking windows is earliest, ties going to the earlier entry in the list, and logs that every account was short.
5. It writes the counter file with the set counts, the current weights and listing, and the taken account's count incremented, writes the worktree's record, then releases the lock.

The counter file also holds each name's weight and the names listed at the last pick, rewritten at every pick; a name it lacks has count 0.
Before the comparison, every account that was not listed at the last pick, or whose weight differs from the stored one, has its count set, whether that raises or lowers it:
its count becomes the lowest count over weight among the other listed accounts that were listed at the last pick under an unchanged weight, times its own weight.
When no such account exists, as on the hub's first pick or after a list change that replaced or reweighted every account at once, every account the rule sets gets count 0.
A count the file kept for a re-added name is overwritten by the set.
So a new, re-added or reweighted account takes its share from the next pick, never catches up on past picks and never waits for the others to catch up,
and an account listed throughout under one weight is never changed, which keeps the counter's spacing.

The counter and the headroom records (D5) are per hub, like the machine layer: the share holds within a hub, and a hub learns of another hub's use of an account only through the figures its own sessions see.

Rationale: a counter keeps the long-run share at the weights, which a hash of the slug cannot do once accounts are skipped.
Count over weight spreads the picks evenly, for example A, B, A for weights 2 and 1, rather than in bursts.
Falling back to the account that is clear of every blocking window soonest keeps work starting when every account is near a limit (D8).

### D4 — The claude engine's launch sets the account's config directory

The claude engine resolves the worktree's directory (D2, D3) when it prepares a session.
It puts the export of `CLAUDE_CONFIG_DIR` into the launch command and the resume command it builds.
No caller, and neither `internal/reedengine` nor `internal/hubgeom`, names the variable.

- The export comes after the `env` module's exports in the pane's launch line, so the recorded directory wins over an `env` module value of `CLAUDE_CONFIG_DIR`.
  With no record and no accounts, the engine exports nothing and an `env` module value stands as today.
- A recorded directory equal to Claude Code's default, `~/.claude`, gets `CLAUDE_CONFIG_DIR` unset in the launch rather than set.
  Setting it to the default would move `.claude.json` from `~/.claude.json` into `~/.claude/`, which loses that account's existing state.
- The variable sits in the launch command itself, so a reed relaunch that replays the stored command keeps the directory it was launched with (D2).
- The claude engine's scrub, `ScrubbedEnv`, also names `CLAUDE_CONFIG_DIR`, so reed's tmux server never carries one.
  A server spawned by a lym process in a pane exported to one account therefore never runs a later pane with no export under that account.
  An operator's shell value of `CLAUDE_CONFIG_DIR` then reaches no lym pane either, in a hub or outside one; the `env` module is the way to set a directory for every pane.
  An operator whose lym sessions ran under a shell value sets the `env` module's `CLAUDE_CONFIG_DIR` to it before the change lands, so those sessions still resume;
  a tmux server already running keeps its environment until it restarts.
- The engine's open launch, `Claude.OpenLaunch` behind `lym shuttle open`, reads and writes the worktree's record by D2's rules,
  and it exports or unsets `CLAUDE_CONFIG_DIR` in its launch and resume lines as the prepared launch does.

Facts this rests on (Claude Code 2.1.322 installed; docs at code.claude.com, read 2026-11-11):

- `CLAUDE_CONFIG_DIR` selects a directory with its own settings, session history and claude.ai login (Authentication docs, "Log in with multiple accounts").
- On Linux and Windows the login is kept in `.credentials.json` in the config directory, by default `~/.claude` and `%USERPROFILE%\.claude` on Windows;
  on macOS it is kept in the Keychain, keyed to the directory (same page, "Credential management").
- Every `~/.claude` path moves under it, transcripts in `projects/` and the session registry included, and so does `.claude.json`, which otherwise sits at `~/.claude.json` (claude-directory docs).

Rationale: the board asks for one place in the claude engine's launch, and `PATTERN-shuttle-provider-seam` keeps provider specifics such as this variable inside `internal/shuttleengine/claudeengine`.

### D5 — Headroom is read from Claude Code's status line input, recorded per account

When the worktree's recorded directory is a listed account's, compared cleaned (D1), the session settings file the claude engine renders and passes with `--settings` carries a `statusLine` command.
The engine resolves that account at launch against the current list and passes the entry's current name and cleaned directory, never the name stored in the record;
with no listed entry for the directory, the settings carry no status line.
An opened session (D4) writes settings only in a hub with a board directory, and those settings carry the same command by the same rule;
an opened session that writes none carries no status line.

The command runs a lym verb with two arguments: that account's name and its config directory, cleaned (D1), as the engine resolved it at launch.
The rendered command names the lym binary through `selfexec.LymBinary` and changes to the worktree's absolute path, which the engine knows at launch, before the verb runs,
so the verb resolves its hub from the worktree's anchor whatever directory Claude Code runs the command in; no hub path argument is added.
The verb refuses a name that does not match `[a-z0-9-]+`, that names no listed account, or whose listed entry's cleaned directory differs from the directory argument:
it writes nothing and prints the line without figures.
It reads the status line JSON on stdin and takes `rate_limits.five_hour` and `rate_limits.seven_day`, each `used_percentage` and `resets_at`.
It drops a window whose `used_percentage` lies outside 0 to 100, or whose `resets_at` lies more than the window's own length, five hours or seven days, past now.
It writes the rest into `<Bolt>/.lym/claude/usage/<account>.json` under an `internal/lock` write lock, through `internal/state`.
The file holds the directory argument beside the windows.
When the stored file names another directory, the incoming windows replace the whole file.
Otherwise it merges them window by window:

- an incoming window with a later `resets_at` than the stored one replaces it;
- one with the same `resets_at` replaces it only when its `used_percentage` is higher;
- one with an earlier `resets_at` is dropped.

So an idle session that repeats older figures never lowers what a busier session recorded.
It prints one short line: the account name and each window's used percentage.
An input without `rate_limits` writes nothing and prints the line without figures.

Any shell in the hub can run the verb.
A forged input can set one listed account's figures in one window, raising or lowering its eligibility, until the forged reset: at most five hours on for `five_hour` and seven days for `seven_day`.

A window has headroom when its record is absent, when its `resets_at` has passed, or when its `used_percentage` is below the account's threshold for it.
So an account no session has used yet, or one idle past its reset, is eligible.

In lym's agent panes this status line replaces the operator's own, since settings passed with `--settings` take precedence over user settings, and it does not chain to the operator's command.

Facts this rests on: the status line input carries `rate_limits.five_hour` and `rate_limits.seven_day` with `used_percentage` (0 to 100) and `resets_at` (Unix seconds) since Claude Code 2.1.80.
It carries them only for subscription accounts and only after the session's first API response.
Either window may be absent on its own, and one past its `resets_at` is dropped (statusline docs; the installed 2.1.322's changelog).
No documented route reads an account's usage without a running session; `/usage` is interactive only.

Rationale: the status line is the one documented source that gives the session and weekly figures the operator asks for (board entry, operator 2026-11-11).
The figures go stale while an account is idle, and the reset times bound that staleness: a passed reset reads as free.
Resolving the account by its cleaned directory at launch and passing its name and directory means the verb compares one path, the named entry's, by equality,
and a renamed account, or a name reused for another directory, files its figures under the entry that names the directory now.
A session launched before the change to the list passes the old name and directory until its next launch, so the verb refuses its figures when that name left the list or names another directory now.
The first figures filed for a name moved to another directory replace the old directory's file whatever its resets, so the new directory is never judged on the old one's figures.

### D6 — Every lym reader of Claude's config directory reads the right directory

The readers that today take `$HOME/.claude` read the right config directory:

- A reader told the worktree, such as the live session's transcript reads and the installed-plugins lookup, takes the worktree's recorded directory.
  The resume check resolves its directory by D2's resume rules; its live-holder test reads the registries of every known directory, as the next bullet's id-only read does (D2).
- A reader told only a session id, such as the prober's liveness read, looks the id up in the session registry of every known directory; session ids are UUIDs, so one match is the session's.
  The liveness read takes the entries of every known directory's registry as one set.
  A known directory with no registry directory contributes no entry and is no doubt; a registry directory or entry that exists but cannot be read is doubt.
  The answer then follows the rule one registry follows: any live entry reads alive, dead needs a dead entry and no doubt, and anything else reads unproven.
- The engine's `LocateSessions`, which serves the session-end usage recorder (`shedadapters`' transcript fill, told the session's worktree) and `tools/tokencount`,
  locates a run's sessions by reading the `projects/` directory of every known directory, each once, never through the worktree's record.
  It returns a session id found in two of them once.
  `lym report` reads no transcripts: it prices runs from the usage records that recorder writes.

The known directories are the listed accounts' directories, the directories of the hub's worktree records that still exist, the `env` module's `CLAUDE_CONFIG_DIR` when set, the process's own `CLAUDE_CONFIG_DIR` when set, and the default `~/.claude`.
When `claude.accounts` fails to load, a reader logs the refusal and reads the remaining known directories, so a bad list never hides the sessions recorded on the other directories.
History on an account removed from the list and from every record is no longer read.

The known directories reach each reader as told input; the engine derives no hub geometry (`PATTERN-cwd-resolution`, Told geometry).
Every claude engine that reads or searches the known directories is built for the hub with its config `Dirs`, `Board` and `Anchor` both, so it loads `claude.accounts` and the `env` module:
the engine that prepares, opens and resume-checks a session, the one behind the id-only liveness read, and the readers `shuttleengines.RecordReaders` returns, which takes those `Dirs`.
A call site that builds such an engine with `Dirs` lacking `Board` tells the hub's `Dirs` instead.
The hub's worktree anchors reach each such engine as a told list of absolute paths its caller resolves, and the engine reads each anchor's record.
A reader run outside a hub has neither, no machine layer and no records, and reads the process's `CLAUDE_CONFIG_DIR` and the default alone.

A reader with no record to go by takes the `env` module's value, then the default, as a lym pane does, since reed's server carries no `CLAUDE_CONFIG_DIR` (D4).

The hook-driven readers already take `transcript_path` from the hook payload and need no change.

Rationale: under a second config directory the transcripts, the session registry and the plugin list live there (D4), so a reader of `~/.claude` alone loses every session on that account: resume, liveness, usage records and cost reports.
The `env` module's value reaches only panes, gate tiers and Drift steps, never lym's own process, so the readers read it from the module.

### D7 — The prime's and each beam's long-lived sessions follow the same per-worktree rule

The prime worktree and each beam worktree record their directory like a task worktree (D2).
The orch, the dobby and any other session there run under it.
A beam's account is independent of the prime's.
Their sessions started before this change have their transcripts in the directory they ran under, and the first launch in the worktree, a resume or a fresh one, records that directory (D2).
They keep it until the operator changes the record.

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

- A worktree's account is sticky while its directory exists, the prime's and beams' included (D2, D7).
  The operator changes it by writing the record with another directory, which a record holding only its `config_dir` key does, the record being read tolerantly;
  a removed record is written again with the directory that holds the transcripts of the worktree's resumable sessions, so removal alone moves only a worktree with none.
- The status line does not chain to the operator's own command (D5).

## Open risks

- The headroom view and the share are per hub (D3), and this machine runs several hubs; a hub that rarely uses an account can pick it while another hub has spent it, until its own session sees the figures.
- Claude Code may change the status line input's field names; the verb then writes nothing and every account reads as having headroom, so the pick falls back to the weights alone.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - counter-bursts-readded-or-reweighted (class design, severity MEDIUM, open in rounds 2, 3)
  - d6-liveness-no-combining-rule (class design, severity MEDIUM, open in rounds 3)
  - fresh-launch-session-sources-incomplete (class design, severity BLOCKING, open in rounds 3)
  - gone-recorded-directory-empty-list (class consistency, severity MEDIUM, open in rounds 3)
  - resume-live-holder-reads-default-registry (class design, severity BLOCKING, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: a `claude.accounts` list in the machine layer loads with each entry's name, directory, weight and windows; a duplicate name, a relative path, two entries naming one directory after cleaning, a non-positive weight, an unknown window or a threshold outside 1 to 100 is refused.
- D1: a non-empty `claude.accounts` from a tracked `claude.yaml` is refused, also when the machine file lists accounts, and the refusal names `lym config claude --layer machine`.
- D1: with `claude.accounts` empty and no record, a session's launch command and resume command neither set nor unset `CLAUDE_CONFIG_DIR`, and a resume launch writes no record.
- D2: a second session started in a worktree after the first uses the recorded directory, without a new pick.
- D2: a record whose account was removed from the list, its directory still present, is kept and its directory exported.
- D2: a fresh launch with a record whose directory is gone, and no transcript of the worktree in any known directory, picks again and replaces the record;
  a resume launch with such a record is refused, naming the directory;
  with `claude.accounts` empty, a fresh launch and a resume launch with such a record are both refused, naming the directory and the way forward, and neither exports nor writes anything.
- D2: a resume launch in a worktree with no record, with `claude.accounts` listing an account, its transcript in the default directory, records the default directory and advances no count.
- D2: a fresh launch in a worktree with no record, whose resumable sessions' transcripts sit in the default directory, records the default directory and advances no count.
- D2: a fresh launch in a new worktree at a path whose earlier worktree left transcripts in the default directory, with no session of its own, picks.
- D2: a resume launch with no record, its transcript found in no known directory, writes no record.
- D2: a resume launch in a worktree with no record, with `claude.accounts` listing an account, its transcript under the `env` module's `CLAUDE_CONFIG_DIR`, records that directory.
- D2: a resume launch with no record, its transcript in a listed non-default account's directory, passes the resume check and records that directory.
- D2: a fresh launch in a prime whose orch state record names a session whose transcript is in the default directory records the default directory and makes no pick.
- D2: a fresh launch in a worktree with no record, whose only resumable session id sits in a PR-Review or sampler record, its transcript in the default directory and its shuttle run directory gone, records the default directory and makes no pick.
- D3: with weights 2 and 1 and both accounts eligible, six picks give the first account four and the second two, never two of the second in a row.
- D3: an account with a window at or above its threshold, its reset in the future, is passed over; the same account with its reset passed is eligible.
- D3: an account whose directory has no `.credentials.json` is never picked, by the weights or the fallback; with none having one, the launch fails naming the directories.
- D3: with every candidate short, the pick takes the candidate whose latest blocking reset is earliest;
  a candidate with two blocking windows is judged by the later of them, and a tie goes to the earlier entry.
- D3: an account added to a list with existing counts has its count set to the lowest count over weight among the other accounts listed at the last pick under an unchanged weight, times its weight, never left at zero;
  an account whose weight was lowered, and a re-added account with a kept count, has its count set to that value the same way, whether that raises or lowers it; an account listed throughout under one weight is never changed.
- D3: after a re-added account's count is set, or a reweighted one's, it is picked only while its count over weight is no higher than that of every other eligible account,
  and an account that gained weight takes no pick before its count over weight reaches the lowest of the others.
- D3: two sessions starting at once in one worktree with no record end with one record and one counter increment.
- D3: a resume and a fresh launch starting at once in one worktree with no record end with one record.
- D4: a session in a worktree whose recorded directory is not the default gets a launch command and a resume command that export `CLAUDE_CONFIG_DIR` as that directory, after any `env` module value.
- D4: a worktree whose recorded directory is `~/.claude` gets a launch command that unsets `CLAUDE_CONFIG_DIR`.
- D4: a session opened with `lym shuttle open` in a worktree recorded on a non-default directory gets launch and resume lines that export `CLAUDE_CONFIG_DIR` as that directory.
- D4: a tmux server spawned from a process whose environment holds `CLAUDE_CONFIG_DIR` carries none of it.
- D5: the status line verb, given an account name, its directory and input with both windows, writes that account's usage file with both windows' percentage and reset; given input without `rate_limits`, it writes nothing.
- D5: the verb given a name that does not match `[a-z0-9-]+`, a name no listed account has, or a directory other than the named entry's, writes nothing.
- D5: after a name moves to another directory, the first status line input for it replaces the stored windows whatever their resets.
- D5: a window with `used_percentage` outside 0 to 100, or with `resets_at` more than its window's length past now, is dropped, and the other window is written.
- D5: a stored window is replaced by one with a later reset or a higher percentage at the same reset, and kept against one with a lower percentage at the same reset or an earlier reset.
- D5: a session whose worktree's recorded directory is a listed account's gets session settings carrying the status line command with that entry's current name and directory, after the account was renamed in the list too;
  with no listed entry for the directory, the settings carry none; an opened session's settings follow the same rule.
- D5: an account whose usage file lacks a window it lists, or has none, reads as having headroom in that window.
- D6: the engine's live-session transcript read, resume check and plugin lookup for a worktree recorded on a fake directory find the files in that directory and not under `~/.claude`.
- D6: the resume check refuses a resume of a session registered as held by a live process only in a listed non-default account's registry, also when the check resolved another directory, and passes one whose registry entry there is dead.
- D6: the liveness read of a session id registered only under a fake account directory finds the session.
- D6: a session registered dead in a listed account's registry, the default directory holding no registry directory, reads dead; one registered alive in any known directory reads alive.
- D6: the session-end usage record of a session run on a listed non-default account carries its transcript's figures.
- D6: `tools/tokencount` counts a landed run's sessions under every known directory, a listed non-default account's and the `env` module's included, with the run's worktree and record gone, and counts a session id found in two of them once.
- D6: a record reader built for a hub finds a session under a listed account's directory with the session's worktree and its record gone.
- D7: a prime or beam worktree's orch session and dobby session run under the worktree's recorded directory.
- D8: a session that hits a usage limit keeps its recorded `CLAUDE_CONFIG_DIR` across the wait and its resume.
- Review: D9's manual check on the real second account is written in the task's landing notes with its outcome.
