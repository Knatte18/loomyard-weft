# Decision record — claude-account-pick-simple

## Goal

Spread Claude usage over the Claude accounts logged in on one machine, as simply as it can be built.
Each worktree runs its Claude sessions under one account, picked by weight and headroom when the worktree is made on the machine,
pinnable per hub or for the prime and beams, and switched to another account when its own runs out.
The whole account policy is machine-local; the repository's tracked config and git hold none of it.
The estimate is about 1,300 to 1,800 lines with tests, against the cancelled design's 3,900.

## Scope

In:

- The account list and the pins in the `claude` config module's machine layer (D1).
- One machine-local, hub-wide account file: each worktree's account and the pick counter (D2).
- The pick when a worktree is made on the machine (D3).
- The claude engine's launch resolving the account, exporting its directory and recording it per run (D4).
- Headroom from Claude Code's status line input and from usage-limit hits (D5).
- Bringing a session's newest transcript into the account a resume runs under (D6).
- The automatic switch on a usage limit, and the explicit switch verb (D7).
- Lym's readers of Claude's config directory reading the right directories (D8).
- The prime and the beams (D9).
- Tests on fake config directories, and a manual check on a machine with two accounts (D10).

Out:

- Logging the accounts in, preparing a config directory, and whether the use is within the accounts' and the organisation's terms: the operator's.
- Any engine other than Claude.
- Standalone runs, outside a hub: no machine layer, so no accounts and today's behaviour, but for reed's server no longer carrying a shell `CLAUDE_CONFIG_DIR` (D4).
- How a run goes on after `lym adopt` moves it to another machine: the adopt work owns that.

## Decisions

### D1 — Accounts and pins are machine-layer keys of the `claude` module

New keys in `internal/shuttleengine/claudeengine/config.go`'s template and `ConfigSpec`, each classified `CategoryMachine`:

- `accounts`: a list; each entry holds `name`, `config_dir`, `weight` and `windows`.
  - `name` matches `[a-z0-9-]+` and is unique.
  - `config_dir` is absolute, a leading `~/` expanded to the home directory; directories are compared expanded and cleaned, and two entries never name one directory.
  - `weight` is a non-negative integer; the long-run share of picks is weight over the sum of weights.
    An account of weight 0 is retired: never picked or switched to, but its directory stays known to every reader (D6, D8), so its sessions are still found.
  - `windows` maps a window name of the closed set `five_hour`, `seven_day` to the used percentage, 1 to 100, at or above which the account is passed over in that window.
    An account is judged only by the windows it lists; there is no default threshold.
- `account_pins.hub`: an account name; every worktree of this hub runs under it.
- `account_pins.beams`: an account name; the prime and every beam worktree run under it, and with them every orch and dobby.
  A hub pin wins over a beams pin.

Template defaults are `[]` and `""`.
The module's load refuses a non-empty value of any of these keys read from a tracked layer, the hub's or the worktree's `claude.yaml`, also when the machine layer holds the key, naming the key and `lym config claude --layer machine`.
It refuses an invalid list, and a pin naming no listed account or a retired one, the same way, and the session launch fails; a bad value never degrades to no accounts.

With fewer than two accounts of positive weight, no pick and no switch runs; a pin still applies, as the operator's own choice.
With no account and no pin, no launch exports anything; only D4's scrub differs from today.

Rationale: the accounts differ by machine (board, operator 2026-11-15), and account details never go into a pushed file (operator, 2026-11-07).
The machine layer is the one never-tracked layer `PATTERN-config-layers` opens to a key, and it is per hub; a hub is one repository, so the hub pin is the board's "a whole repository to one account" on this machine.

### D2 — One machine-local account file per hub, a cache

`<Bolt>/.lym/claude/accounts.json`, named through `internal/lymdirs` and written through `internal/state` under one `internal/lock` write lock, holds:

- `worktrees`: worktree name to account name, the worktree's entry;
- `counts`, `weights`, `listed`: the pick counter (D3).

An entry holds an account's name, never its directory: a launch maps the name through the current list, and an entry whose name the list lacks reads as no entry.
A pick prunes entries whose worktree directory no longer exists in the hub.
Nothing of the file goes into git: it is a cache in `PATTERN-lym-files`' sense, and a fresh or wiped file reads as empty, so every worktree runs as today or under its pin, and a resume still finds its transcript by D6.

Rationale: the cancelled design's per-worktree record grew mostly from the transcript search its loss forced (board, "Keeping it small").
Keying by name and bringing transcripts along on resume (D6) makes a lost or changed entry harmless.
The file decides no run's course, only which login a session uses, and is meaningless on another machine, so the adopt design's state-in-git rule does not reach it.

### D3 — The pick: a smooth weighted counter that passes over an account short of headroom

The pick runs when a task or beam worktree is made on the machine: the task run's Worktree-Create step, a new beam's creation, and each worktree `lym adopt` adds.
It replaces any entry the file holds for that name, so a worktree recreated under a used name is weighed afresh.
It does not run for a worktree a pin covers, nor for the prime (D9).

Under the file's lock it:

1. Takes as candidates the listed accounts of positive weight that are logged in: on Linux and Windows those whose `config_dir` holds `.credentials.json`, stat only;
   on macOS, where the login is in the Keychain, every such account.
   With no candidate it removes the name's entry, logs it, and the worktree runs as today.
2. Takes as eligible the candidates with headroom in every window they list (D5), and among them the lowest count over weight, ties to the earlier entry.
3. With none eligible, takes the candidate whose latest blocking reset is earliest, ties to the earlier entry, and logs that every account was short.
4. Writes the entry and the taken account's incremented count.

Before step 2, an account not listed at the last pick, or whose weight changed, has its count set to the lowest count over weight among the accounts listed then under an unchanged weight, times its own weight; with no such account, to 0.
So a new or reweighted account takes its share from the next pick and never catches up on past picks.

The counter, the entries and the usage records are per hub, like the machine layer.

Rationale: a counter keeps the long-run share at the weights where a slug hash cannot once accounts are skipped, and count over weight spreads picks (A, B, A for weights 2 and 1).
Picking at the worktree's making is the board's moment ("every time a worktree gets work on a machine where it has no account yet") and leaves worktrees made before the change on today's directory.

### D4 — The claude engine's launch resolves the account, exports it and records it in the run

At every launch it builds, a fresh one, a resume and `OpenLaunch`'s lines, the claude engine resolves the worktree's account:
the hub pin, else the beams pin for the prime or a beam worktree, else the file's entry (D2), else none;
a pinned account with a `blocked_until` in the future (D5) is skipped in favour of the entry, so a pinned worktree can run elsewhere while its pin is out (D7).
The runner tells the engine the worktree's name, whether it is the prime or a beam worktree, and the hub's account file path, as told geometry.

- An account whose directory is not `~/.claude`: the launch and resume commands export `CLAUDE_CONFIG_DIR` as that directory, after the `env` module's exports, so it wins over an `env` value.
- An account on `~/.claude`: they unset `CLAUDE_CONFIG_DIR`, since setting it to the default moves `.claude.json` into `~/.claude/` and loses that account's state.
- No account: they neither set nor unset it, and the `env` module's value or the default stands.

The launch's effective directory is the exported one, else the `env` module's value, else `~/.claude`.
`Prepare` and `OpenLaunch` write it into the run directory with the listed account whose cleaned directory equals it, if any.
That launched account is the session's account wherever D5, D7 and D8 need one, also for a worktree with no entry that runs on a listed directory.

The engine's scrub list, which reed's server spawn strips, gains `CLAUDE_CONFIG_DIR`, so a tmux server spawned from a pane on one account never carries it into a pane with no export.
An operator's shell `CLAUDE_CONFIG_DIR` then reaches no lym pane; the `env` module is the way to set one for every pane.

A reed relaunch replays the stored command, so it keeps the account it was launched under until a launch the engine builds starts the strand again.

Facts (Claude Code 2.1.327 installed; code.claude.com, read 2026-11-18): `CLAUDE_CONFIG_DIR` selects a directory with its own settings, session history and claude.ai login (Authentication, "Log in with multiple accounts");
on Linux the login is `<dir>/.credentials.json`, on macOS a Keychain entry keyed to the directory (same page, "Credential management");
transcripts sit in `<dir>/projects/<project>/<session>.jsonl`, and `.claude.json` moves under the directory (claude-directory page).

Rationale: one place in the engine, as `PATTERN-shuttle-provider-seam` keeps the variable in `internal/shuttleengine/claudeengine`; reading the account at every launch is the board's "every spawn reads the worktree's account at that moment".

### D5 — Headroom comes from the status line input and from usage-limit hits, recorded per account

A session whose launched account (D4) is a listed one gets a `statusLine` command in the settings the engine passes with `--settings`.
It runs a lym verb with the account's name, through `selfexec.LymBinary`, from the worktree's directory.
The verb refuses a name no listed account has, and writes nothing.
It reads `rate_limits.five_hour` and `rate_limits.seven_day` from stdin, each `used_percentage` and `resets_at`,
drops a window whose percentage lies outside 0 to 100 or whose reset lies more than its window's length ahead,
and merges the rest into `<Bolt>/.lym/claude/usage/<account>.json` under a write lock, through `internal/state`:
a window with a later reset replaces the stored one, one with the same reset replaces it only with a higher percentage, one with an earlier reset is dropped.
It prints one short line: the account name and each window's percentage.
Input without `rate_limits` writes nothing.

A shuttle run or orch session that hits a usage limit, as the engine's turn-end classification reports it with its reset (`internal/shuttleengine/claudeengine/activity.go`, `internal/shuttleengine/claudeengine/usagelimit.go`),
records `blocked_until` on its launched account's usage file: that reset, or 30 minutes on when the message names none, the later of it and a stored one winning.
The status line merge keeps `blocked_until`.

A window has headroom when its record is absent, its reset has passed, or its percentage is below the account's threshold; an account with a `blocked_until` in the future has none in any window.

Bounds: any shell in the hub can run the verb, so forged input can set one listed account's figures in one window until the forged reset, at most five hours or seven days on.
A session opened with `lym shuttle open` is not waited on by shuttle, so its limit hits record no `blocked_until`; its status line still records figures.

Facts (statusline page, code.claude.com, read 2026-11-18): the input carries `rate_limits.five_hour` and `rate_limits.seven_day` with `used_percentage` 0 to 100 and `resets_at` in Unix seconds,
only for subscription accounts and only after the session's first API response; either may be absent.
Claude Code 2.1.327's `claude --help` and `claude auth` offer no non-interactive usage read, and `/usage` is interactive only.
Settings passed with `--settings` rank above user settings, so the lym status line replaces the operator's own in lym panes, unchained.

Rationale: the status line is the one documented source of the session and weekly figures (operator, 2026-11-11); a limit hit is the surest sign an account is out, and it carries its reset.

### D6 — A resume first brings the newest copy of its transcript into the account it runs under

The claude engine's resume check, which shuttle runs before `Prepare` (`internal/shuttleengine/run.go`) and which the orch's resume reaches through the same runner, first resolves the account by D4.
It then finds `<session>.jsonl` under the worktree's project in every known directory (D8), the resolved one included.
When the most recently written copy is not the resolved directory's, it copies that file, with its `<session>/` directory when present, over the resolved directory's, leaving the source in place.
It then stats and checks as today: found nowhere, it refuses with its no-transcript error, and the caller starts fresh or fails as it does today.
Its live-holder test reads the registries of every known directory as one set (D8).
`OpenLaunch` brings the newest copy in the same way when it builds a resume line.

Facts: Claude Code documents no move of a session between config directories.
The Claude API's preserved-thinking page (platform.claude.com, read 2026-11-18) says thinking blocks of several 5.x models are honoured only in the organization that produced them and are dropped elsewhere with no error.
So a copied transcript resumes under another account, losing the earlier turns' thinking but no message.
The 400 "invalid signature" error is the prefix check and compares no account; it fires when the tools or system prompt differ, which the two accounts sharing `plugins/` avoids (D10).

Rationale: the board asks the Discussion to check whether a copied transcript resumes; the API documents that it does, so the copy is the switch mechanism and no session-id search through owner records is needed.
Taking the newest copy rather than any copy keeps every turn when a worktree switches back to an account that holds an older copy.

### D7 — A worktree whose account runs out switches; any session in it can switch it by hand

When a shuttle run's session hits a usage limit (D5), its run's `Wait` asks the engine, through an engine-neutral optional capability beside the usage-limit reader in `internal/shuttleengine/hold.go`, to switch the worktree.
Under the account file's lock the engine reads the session's launched account (D4) and:

- when the worktree now resolves (D4) to another account with headroom, as after a sibling session's switch, makes no new switch;
- otherwise takes, by D3's steps 1 and 2, an eligible candidate other than the launched account, writes it as the worktree's entry and increments its count;
- with none eligible, changes nothing.

A pinned worktree switches too: its pinned account now has a `blocked_until`, so D4 resolves to the new entry until that time passes, and the next launch after it returns to the pin.

When the worktree then resolves to another account, the run's `Wait` relaunches its strand inside the same run as a resume of the same session id, the way shuttle's own respawn replaces a strand:
no stopped outcome is recorded, the waiter keeps its run, and the run ends once, with one session-end usage record.
D6 carries the transcript over.
Otherwise the limit is handled as today.
The orch watcher (`internal/orchengine/usagelimit.go`) asks the same switch and, on a switch, restarts the orch through its resume path instead of waiting.

The verb `lym claude account set <account>`, run in a worktree, writes that worktree's entry; `lym claude account show` prints the worktree's resolved account, the pins and each account's headroom.
A set takes effect at the worktree's next engine-built launch; running strands keep their account until then.
Bound: any session in the worktree, an agent's included, can run it; it changes only the machine-local entry, refuses an account the list lacks or has retired, and cannot override a pin that is not out.

Rationale: the board replaces the previous record's D8 ("a usage limit is waited out only when no account of the machine has headroom") and asks for an explicit switch verb.
Relaunching inside the run keeps the producer's wait whole; a stop through shuttle's stop verb would end the run as stopped.

### D8 — Lym's readers of Claude's config directory read the right directories

The known directories are the listed accounts' directories, retired ones included, the `env` module's `CLAUDE_CONFIG_DIR` when set, and `~/.claude`, expanded and cleaned, each once; outside a hub, the process's own `CLAUDE_CONFIG_DIR` and `~/.claude`.

- `LocateSessions` (`internal/shuttleengine/claudeengine/transcriptread.go`), behind the session-end usage recorder and `tools/tokencount`, reads the worktree's project in every known directory
  and returns a session id found in several once, the most recently written copy, which by D6 holds every turn.
- The readers of a live session's transcript, the live transcript read, the fork audit, the outstanding-forks read and the session-name drift read, read the run's launched directory (D4), not the worktree's current resolution, since a switch or a set leaves running strands where they are.
  A run with no launched directory recorded, from before the change, reads under the worktree's resolution, then the other known directories.
- The installed-plugins lookup takes the launch's effective directory.
- The id-only reads, `SessionHolder` and the prober's liveness read, take the registries of every known directory as one set:
  any live entry reads alive, dead needs a dead entry and no unreadable registry, anything else reads unproven.
- Every claude engine that reads the known directories is built with the hub's config `Dirs`, `Board` and `Anchor` both, so it loads the accounts and the `env` module:
  the one that prepares and resume-checks, the session namers reed's watchdog takes, and the record readers handed outside the engines; a builder lacking them today is told the hub's `Dirs`.
- A `claude.accounts` that fails to load is logged, and the readers read the remaining known directories.

Hook-driven readers take `transcript_path` from the hook payload and need no change.

Rationale: a session on a second account keeps its transcript and registry entry there, so a reader of `~/.claude` alone loses its usage records, liveness and cost; the `env` value reaches only panes, so the readers read it from the module.
An account removed from the list drops out of the known directories; retiring it with weight 0 keeps its history read.

### D9 — The prime and the beams follow the per-worktree rule, under the beams pin when set

The prime and each beam worktree resolve their account by D4, the beams pin first.
With no pin, a beam is picked at its creation and at each adopt onto a machine (D3).
The prime is never picked, `lym adopt root` included: with no pin it runs as today until `lym claude account set` or a limit switch (D7) gives it an entry.
Orch and dobby sessions started before the change resume under the resolved account through D6's copy.

Rationale: the board keeps the root "mostly on one account" and puts every orch on one account online under Remote Control while task worktrees spread (operator, 2026-11-15).

### D10 — Tests run on fake config directories; a manual check runs on a machine with two accounts

Automated tests use fake config directories under the test's temporary directory, with an empty `.credentials.json` where a logged-in account is needed, and feed the status line verb fake JSON.
No automated test reads a real account or calls Claude.

The operator wires the second account once, on the machine that has it:
log it in with `CLAUDE_CONFIG_DIR=<dir> claude` and `/login`; link `<dir>/plugins` to `~/.claude/plugins`, so lym's deploy, which syncs plugins into `~/.claude`, serves both and both accounts load the same tools;
link the user-level instruction file and settings as wanted; list both accounts with `lym config claude --layer machine`.
The check starts two task runs, confirms their worktrees' entries and that a transcript lands in each account's `projects/`, then switches one worktree with `lym claude account set` and resumes a session there.

Rationale: the board says a test of the pick and headroom needs a real second account the operator provides, and that the task states how it is wired.

## Constraints

- `PATTERN-shuttle-provider-seam`: `CLAUDE_CONFIG_DIR`, the credentials file, the status line input, the account file, the pick and the copy live in `internal/shuttleengine/claudeengine`;
  shuttle reaches the switch through an engine-neutral optional capability, a CLI verb reaches the engine only through `internal/shuttleengines`, and the in-run relaunch stays inside `internal/shuttleengine`.
- `PATTERN-config-layers`: the new keys are classified `CategoryMachine` in the module's `ConfigSpec` in the same change; they are new, so no reinterpretation.
- `PATTERN-lym-files`: the account file and the usage directory are named in `internal/lymdirs` under the Bolt's `.lym`, as caches a fresh machine starts without; lock paths come from `internal/lock`.
- `PATTERN-tolerant-records`: the account file, the usage files and the run directory's launched-account record are read tolerantly and rewritten through `internal/state`.
- `PATTERN-cwd-resolution`: the engine gets the account file path and the worktree's name and kind as told data, never derives hub geometry.
- `PATTERN-cli-cobra`: `lym claude account` and the status line verb are a cobra subtree under the one root, wired through `internal/cliwire`.
- `PATTERN-engine-parity`: the status line is an engine settings item, listed and never compared, so Codex owes no counterpart.

## Auto-mode assumptions

- A pinned worktree switches on a limit while its pin is out, and returns to the pin after (D4, D7), following the board's "waited out only when no account has headroom".
- The pick runs only at a worktree's making and never for the prime (D3, D9).
- The lym status line replaces the operator's own in lym panes (D5).

## Open risks

- For the run's parent: the beams pin covers every session of a beam worktree, so card implementers that run in a beam worktree stay on the pinned account.
  The board says the workers under orchs and beams spread; this record reads those workers as the task worktrees under them.
  If beam-worktree workers must spread too, the pin has to be by role, orch and dobby, which breaks one account per worktree.
- A copied transcript resumes on the documented API behaviour (D6), not on a run; the manual check (D10) is its first real test, and two accounts with different plugin sets can still fail the prefix check.
- The headroom view is per hub, and several hubs share the accounts; a hub can pick an account another hub has spent until its own sessions see the figures or hit the limit.
- Claude Code may rename the status line fields; the verb then writes nothing and the pick falls back to weights and limit hits.

## Acceptance criteria

- D1: a machine-layer `claude.accounts` list loads; a duplicate name, a relative path, two entries naming one cleaned directory, a negative weight, an unknown window or a threshold outside 1 to 100 is refused.
- D1: a non-empty `accounts`, `account_pins.hub` or `account_pins.beams` in a tracked `claude.yaml` is refused, also when the machine file holds the key, naming `lym config claude --layer machine`; a pin naming no listed account, or a retired one, is refused.
- D1: with one account of positive weight no pick runs at worktree making; a pin naming it is still exported.
- D2: an entry whose account name left the list reads as no entry; a missing or empty account file leaves every launch of an unpinned worktree unchanged from today.
- D2: a pick prunes the entry of a worktree whose directory is gone.
- D3: with weights 2 and 1 and both eligible, six picks give four and two, never the second account twice in a row.
- D3: an account with a listed window at or above its threshold and a future reset is passed over; with its reset passed it is eligible.
- D3: an account whose directory lacks `.credentials.json`, or of weight 0, is never picked; with no candidate the name's entry is removed.
- D3: with every candidate short, the pick takes the one whose latest blocking reset is earliest, a tie to the earlier entry.
- D3: an added or reweighted account's count is set to the lowest count over weight among the unchanged accounts times its weight; an account listed throughout under one weight keeps its count.
- D3: two picks at once for one worktree end with one entry and one increment.
- D3: the task run's Worktree-Create step, a new beam's creation and an adopt adding a beam or task worktree each pick, replacing an entry the name already had; a pinned worktree and the prime are not picked.
- D4: a worktree whose account is on a non-default directory gets launch and resume commands, and `OpenLaunch` lines, exporting `CLAUDE_CONFIG_DIR` as it after any `env` module export.
- D4: an account on `~/.claude` gets `CLAUDE_CONFIG_DIR` unset; no account gets neither.
- D4: a hub pin wins over the entry and over a beams pin; a beams pin applies to the prime and a beam worktree and not to a task worktree; a pinned account with a future `blocked_until` yields to the entry.
- D4: the run directory records the launch's effective directory and the listed account it equals, also for a worktree with no entry on a listed directory.
- D4: a tmux server spawned from a process carrying `CLAUDE_CONFIG_DIR` carries none.
- D5: the verb, given a listed name and input with both windows, writes both; given input without `rate_limits`, or an unlisted name, it writes nothing.
- D5: a window outside 0 to 100, or reset beyond its length ahead, is dropped and the other written; the merge keeps the later reset, the higher percentage at one reset, and `blocked_until`.
- D5: a session whose launched account is listed gets settings carrying the status line command with its name; a session on an unlisted directory gets none.
- D5: a usage-limit hit records `blocked_until` on the launched account at its reset, or 30 minutes on with none, and the account has no headroom until then.
- D6: a resume whose transcript sits only under another known directory copies it, with its session directory, into the resolved directory before the resume check stats it, and passes; the source stays.
- D6: a resume whose resolved directory holds an older copy than another known directory gets the newer copy; a transcript found nowhere is refused as today.
- D6: the resume check refuses a session registered live in any known directory's registry.
- D6: an orch resume in a prime whose account changed resumes its session under the new account and does not launch fresh.
- D7: a shuttle session hitting a limit with another eligible candidate switches the worktree's entry, increments its count, and relaunches inside the same run as a resume of the same session id under the new account; the run records no stopped outcome.
- D7: with no other eligible account the entry is unchanged and the limit is handled as today.
- D7: a pinned worktree whose pinned account hits a limit switches to the entry, and its first launch after the `blocked_until` returns to the pin.
- D7: a worktree with no entry on a listed directory that hits a limit switches to another account, never to its own.
- D7: two sessions of one worktree hitting the limit at once make one switch, and both resume under the new account.
- D7: the orch watcher on a switch restarts the orch through its resume under the new account instead of waiting.
- D7: `lym claude account set` writes the entry, refuses an unlisted or retired account, and the worktree's next launch uses it.
- D8: `LocateSessions` finds a worktree's sessions under every known directory, a retired account's included, and returns a session copied by D6 once, the most recent copy.
- D8: after `lym claude account set` moves a worktree, a running session's fork audit and outstanding-forks read still read its transcript under its launched directory.
- D8: the liveness read, built for a hub, finds a session registered only under a listed non-default directory; dead needs a dead entry and no unreadable registry.
- D8: `tools/tokencount` and the session-end usage record count a session run on a listed non-default account.
- D9: an orch and a dobby in the prime under a beams pin run under the pinned account; `lym adopt root` writes no entry for the prime.
- Review: D10's manual check on the machine with two accounts is written in the task's landing notes with its outcome.

## Notes for the plan writer

- The names `lym claude account`, `accounts.json` and the launched-account record in the run directory are this record's; Plan may rename them within the same shape.
- Shuttle's in-run respawn and the API-error retry in `internal/shuttleengine/apierrorretry.go` are the nearest existing paths for D7's relaunch.
