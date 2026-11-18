# Decision record — claude-account-pick-simple

## Goal

Spread Claude usage over the Claude accounts logged in on one machine, as simply as it can be built.
Each worktree runs its Claude sessions under one account, picked by weight and headroom when the worktree is made on the machine,
pinnable per hub or for the prime and beams, and, unless a hub pin holds it, switched to another account when its own runs out.
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
- Two reed ops for D7's relaunch: ending a strand's process while holding it, and relaunching its pane under the same GUID with a new command and session id.
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
  - `config_dir` is absolute, a leading `~/` expanded to the home directory.
    Directories are compared after expanding, cleaning and resolving links, a directory that does not resolve compared expanded and cleaned,
    and two entries never name one directory.
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

An entry holds an account's name, never its directory: a launch maps the name through the current list.
An entry whose name the list lacks, or whose account is retired, reads as no entry, so its worktree runs under its pin or as today, and D6 carries the transcript.
A pick drops the entries whose worktree name is absent from the hub's current worktree names.
The caller that runs the pick passes a reader of those names as told geometry (D4), and the pick calls it under the file's lock, so a worktree made meanwhile keeps its entry.
Nothing of the file goes into git: it is a cache in `PATTERN-lym-files`' sense, and a fresh or wiped file reads as empty, so every worktree runs as today or under its pin, and a resume still finds its transcript by D6.

Rationale: keying by name and bringing transcripts along on resume (D6) makes a lost or changed entry harmless.
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
   A candidate with a future `blocked_until` is never eligible, whatever windows it lists.
3. With none eligible, takes the candidate whose latest blocking reset is earliest, ties to the earlier entry, and logs that every account was short.
   A future `blocked_until` counts as one of the candidate's blocking resets.
4. Writes the entry and the counter: the counts as set, the current weights and listing, and the taken account's count incremented.

Before step 2, an account not listed at the last pick, or whose weight changed, has its count set to the lowest count over weight among the accounts of positive weight listed then under an unchanged weight, times its own weight; with no such account, to 0.
So a new or reweighted account takes its share from the next pick and never catches up on past picks.

The counter, the entries and the usage records are per hub, like the machine layer.

Rationale: a counter keeps the long-run share at the weights, and count over weight spreads picks (A, B, A for weights 2 and 1).
Picking at the worktree's making is the board's moment ("every time a worktree gets work on a machine where it has no account yet") and leaves worktrees made before the change on today's directory.

### D4 — The claude engine's launch resolves the account, exports it and records it in the run

At every launch it builds, a fresh one, a resume and `OpenLaunch`'s lines, the claude engine resolves the worktree's account:
the hub pin, else the beams pin for the prime or a beam worktree, else the file's entry (D2), else none.
A beams-pinned account with a `blocked_until` in the future (D5) is skipped in favour of the entry when the worktree has one, so a beams-pinned worktree can run elsewhere while its pin is out (D7);
with no entry the pin stands.
A hub pin is never skipped.
The runner tells the engine the worktree's name, whether it is the prime or a beam worktree, and the hub's account file path, as told geometry;
a caller that runs the pick (D3) also tells a reader of the hub's current worktree names (D2).

- An account whose directory is not `~/.claude`: the launch and resume commands export `CLAUDE_CONFIG_DIR` as that directory, after the `env` module's exports, so it wins over an `env` value.
- An account on `~/.claude`: they unset `CLAUDE_CONFIG_DIR`, since setting it to the default moves `.claude.json` into `~/.claude/` and loses that account's state.
- No account: they neither set nor unset it, and the `env` module's value or the default stands.

The launch's effective directory is the exported one, else the `env` module's value, else `~/.claude`.
`Prepare` and `OpenLaunch` write it into the run directory with the listed account whose directory equals it, compared as D1 says, if any.
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
`OpenLaunch` writes such a settings file and passes `--settings` whenever its launched account is listed, also with no creel flag directory.
The command runs a lym verb with the account's name, through `selfexec.LymBinary`, from the worktree's directory.
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

Bounds: any shell that can run the verb can set both windows of every listed account, up to each window's length ahead, and can replace stored figures with a later reset.
Forged figures also steer D7's switch: forged high figures on every other account make a limited worktree switch nothing and wait its limit out, up to each forged window's length;
forged low figures on an exhausted account make switches take it until a session on it hits the limit and the hit's `blocked_until` rules it out.
No forged input writes or clears a `blocked_until`.
A session opened with `lym shuttle open` is not waited on by shuttle, so its limit hits record no `blocked_until`; its status line still records figures.

Facts (statusline page, code.claude.com, read 2026-11-18): the input carries `rate_limits.five_hour` and `rate_limits.seven_day` with `used_percentage` 0 to 100 and `resets_at` in Unix seconds,
only for subscription accounts and only after the session's first API response; either may be absent.
Claude Code 2.1.327's `claude --help` and `claude auth` offer no non-interactive usage read, and `/usage` is interactive only.
Settings passed with `--settings` rank above user settings, so the lym status line replaces the operator's own in lym panes, unchained.

Rationale: the status line is the one documented source of the session and weekly figures (operator, 2026-11-11); a limit hit is the surest sign an account is out, and it carries its reset.

### D6 — A resume first brings the newest copy of its transcript into the account it runs under

The claude engine's resume check, which shuttle runs before `Prepare` (`internal/shuttleengine/run.go`), after it in D7's relaunch, and which the orch's resume reaches through the same runner, first resolves the account by D4.
It then finds `<session>.jsonl` under the worktree's project in every known directory (D8), the resolved one included.
When the most recently written copy is not the resolved directory's, it copies that file, with its `<session>/` directory when present, over the resolved directory's, leaving the source in place.
It then stats and checks as today: found nowhere, it refuses with its no-transcript error, and the caller starts fresh or fails as it does today.
Its live-holder test reads the registries of every known directory as one set (D8).

The resume check reports whether it copied.
On the start path the runner carries that fact into the launch and writes it into the new run directory beside the launched-account record (D4) after the directory is created;
in D7's relaunch the relaunch writes it.
A caller that retries a resume after a live-holder refusal, as the orch's resume does, carries the first check's report into the retry, since that check copied before its holder test refused.
A resume whose check copied the transcript, and whose first turn ends on an API error other than a usage limit, is a refused resume.
It starts the session fresh, as the board's "otherwise the session starts fresh" says:
a shuttle run through D7's in-run relaunch with a fresh launch, the orch through the fresh start the orch watcher is told (D7).
`retryAPIError` neither counts nor re-sends that turn end.

Facts: Claude Code documents no move of a session between config directories.
The Claude API's preserved-thinking page (platform.claude.com, read 2026-11-18) says thinking blocks of several 5.x models are honoured only in the organization that produced them and are dropped elsewhere with no error.
So a copied transcript resumes under another account, losing the earlier turns' thinking but no message.
The tools must match between the two directories: the plugins, the `enabledPlugins` in `settings.json` and the MCP servers in `.claude.json` (D10).

Rationale: the board asks the Discussion to check whether a copied transcript resumes; the API documents that it does, so the copy is the switch mechanism.
Taking the newest copy rather than any copy keeps every turn when a worktree switches back to an account that holds an older copy.

### D7 — A worktree whose account runs out switches; a process in it can switch it by hand

When a shuttle run's session hits a usage limit (D5), its run's `Wait` asks the engine, through an engine-neutral optional capability beside the usage-limit reader in `internal/shuttleengine/hold.go`, to switch the worktree.
Under the account file's lock the engine reads the session's launched account (D4) and:

- under a hub pin, changes nothing;
- when the worktree now resolves (D4) to another account with headroom, as after a sibling session's switch, makes no new switch;
- otherwise takes, by D3's steps 1 and 2, an eligible candidate other than the launched account, writes it as the worktree's entry and writes the counter as D3's step 4 does, with the reset counts, the current weights and listing, and its count incremented;
- with none eligible, changes nothing.

Only the beams pin switches on a limit.
Its pinned account now has a `blocked_until`, so D4 resolves the limited worktree to its new entry until that time passes, and its first launch after it returns to the pin.
The switch writes the entry for the limited worktree only; another worktree on the beams pin with no entry stays on the pin.
A worktree under a hub pin waits the limit out as today.

When the worktree then resolves to another account, the run's `Wait` relaunches its strand inside the same run as a resume of the same session id.
Shuttle has no in-run relaunch today; this one is new in `internal/shuttleengine`, in this order:

1. Shuttle's stop verb, in a new relaunch mode, marks the run record relaunching with the time of the mark,
   then ends the limited session's process through reed without removing its strand, which keeps its GUID and is held from reed's reconcile and resume until step 4.
   It records no outcome, so `Wait` keeps its run, and every reader of the record reads it, within the mark's bound, as a live running run while its strand is down (PATTERN changes).
   The orphan sweep keeps a run directory whose record reads relaunching.
   A failure to end the process releases reed's hold and clears the mark back to running over the live strand, and the limit is then handled as today.
2. The engine's `Prepare` builds the relaunch spec's launch: `ResumeSessionID` set to the session id, the run's resolved skills kept, so the appended system prompt is unchanged,
   and in place of the run's task prompt one continue line telling the session its account was switched and to carry on where it stopped.
   A relaunch launch carries no skill-load turn, since the resumed transcript holds the loads: its first turn is the continue line, and its `PromptLine` is empty.
   So the run directory's launched-account record (D4) and the settings file with its status line (D5) name the new account, and so does the launch it returns.
3. The resume check runs (D6): it carries the transcript over, and its live-holder test no longer sees the ended process.
   The relaunch writes whether it copied into the run directory (D6).
4. In one hold of the record's lock, the run record takes the launch's session id and `PromptLine`, its started flag clears,
   and its prompt offset is set to the events file's size at that moment, so neither the task pointer nor the limited turn end is read again.
   Only then does reed replace the strand's stored `Cmd`, `ResumeCmd` and session id with the launch's and relaunch its pane under the same GUID;
   since the GUID never changes, the record names the new launch before its pane exists.
   The relaunch's startup wait decides on the pane's capture alone, dismissing a trust gate as the start path does, so only a ready pane marks the run started.
   It runs with the events proof off, as `Wait`'s own probe of a not-yet-started run does:
   the run's events file (`RunState.EventsPath`) is kept and already holds the limited session's session-start and turn-start lines,
   so `awaitStartup`'s `checkLivenessTick(..., eventsProof=true)` short-circuit (`turnStartedAfterSessionStart`, `internal/shuttleengine/wait.go`) would mark the pane started before it booted.
   A stale session-start line satisfying the line half is already accepted by `checkLivenessTick`.
   A pane that never reaches ready ends the run as the start path's startup wait ends it: as a strand that died, or as a timeout when the run's own deadline passes first.
   Once the pane is started, any `PromptLine` is sent as `launch` sends it; then the mark clears back to running.

Every record write of the relaunch, the mark, step 4's write, the clear and the died end, re-reads the run record in one hold of the record's lock and leaves a record a stop made terminal as it is;
both stop forms change to read and write the record in one hold of that lock too.
When a stop has made the record terminal, the relaunch removes the strand instead of writing.
Every reader of a run record that tests for a running run reads a relaunching record within its bound as a live running run, through one predicate on the record;
past the bound, until an attach or `Wait` applies it, the predicate reads the record as a running run whose waiter is gone.
A reader that reports a run's activity or session state reports a relaunching run as relaunching, with no turn-end facts,
since its prompt offset precedes the limited turn end until step 4's write.
The stop verb removes a strand held for a relaunch whatever its pane state,
and `Runner.StopStepRuns` (`internal/shuttleengine/stepstop.go`), which skips a strand that is not removable before it reaches the stop verb, stops a relaunching run, within its bound or past it, whatever its pane state too.
Every caller holding the strand's GUID, a stop, an attach or a reclaim, reaches the relaunched run through it, since the GUID never changes.
A check refused for no transcript runs steps 2 to 4 again with a fresh launch: a new session id, the run's skills with their load turn, and the run's prompt.
Any other refusal, or a failure in steps 2 to 4, ends the run as a strand that died, removing its strand, done when every output file exists.
The mark is bounded at twice the startup timeout since the mark.
Past the bound, an attach or `Wait` that reads a relaunching record whose strand is not live ends the run the same way,
and one that reads it over a live strand clears the mark to running and attaches, the record already naming the new launch;
the attach's pending-prompt delivery (`DeliverPendingPrompt`) then types a `PromptLine` the dead `Wait` never sent.
So a `Wait` process that dies anywhere in steps 1 to 4 leaves no run read as live past the bound.
The run ends once, with one session-end usage record.

A usage-limited turn end the switch handles is neither counted toward `APIErrorRetries` nor re-sent by `retryAPIError`.
When the relaunched strand starts, the API-error watch clears its count and pending turn end, its cursor kept past the limited turn end.
With no switch, the limit is handled as today, `retryAPIError` counting and re-sending it.

The orch watcher (`internal/orchengine/usagelimit.go`) asks the same switch.
It is told two capabilities backed by `orchcli`, both running the body of the orch start path's relaunch branch (the `StartRelaunch` case in `internal/orchcli/start.go`) directly,
skipping `DecideStart` and the wait for the watcher to go, since the live limited strand and the calling watcher would decide that the orch is already running:
a restart withdraws the orch's pending mail, removes the old orch strand, launches through `launchOrch` with the session resumed, registers the new process,
and saves the state reset onto the new strand GUID, spawning no second watcher;
a fresh start does the same with the recorded session id cleared, so `launchOrch` launches fresh.
On a switch the watcher restarts the orch through the first instead of waiting, and on a refused resume (D6) starts it fresh through the second;
either way it carries on over the new strand.

The verb `lym claude account set <account>`, run in a worktree, writes that worktree's entry; `lym claude account show` prints the worktree's resolved account, the pins and each account's headroom.
A set takes effect at the worktree's next engine-built launch; running strands keep their account until then.
A set under a hub pin is refused, naming the pin and `lym config claude --layer machine`; under a beams pin it writes the entry, which takes effect while the pin is out.
Bound: any process that can run lym from a worktree's directory, an agent's included, can set that worktree's entry;
it changes only the machine-local entry, refuses an account the list lacks or has retired, and cannot override a hub pin or a beams pin that is not out.

Rationale: the board replaces the previous record's D8 ("a usage limit is waited out only when no account of the machine has headroom") and asks for an explicit switch verb.
Under a hub pin the pinned account is the only one open to the hub, as the board's "this hub only on private account 1" says, so waiting satisfies that line.
Relaunching inside the run keeps the producer's wait whole.

### D8 — Lym's readers of Claude's config directory read the right directories

The known directories are the listed accounts' directories, retired ones included, the `env` module's `CLAUDE_CONFIG_DIR` when set, and `~/.claude`, compared as D1 says, each once; outside a hub, the process's own `CLAUDE_CONFIG_DIR` and `~/.claude`.

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

The operator wires the second account once, on OSL-1046, the machine the board gives both accounts:
log it in with `CLAUDE_CONFIG_DIR=<dir> claude` and `/login`; link `<dir>/plugins` to `~/.claude/plugins`, so lym's deploy, which syncs plugins into `~/.claude`, serves both;
link `<dir>/settings.json` to `~/.claude/settings.json`, so both carry the same `enabledPlugins`, and copy the MCP servers of `~/.claude.json` into `<dir>/.claude.json`, so both accounts load the same tools (D6);
link the user-level instruction file as wanted; list both accounts with `lym config claude --layer machine`.
The operator runs the check on OSL-1046 after the task lands on this run's owner machine, neve, and the change reaches OSL-1046.
The check starts two task runs, confirms their worktrees' entries and that a transcript lands in each account's `projects/`, then switches one worktree with `lym claude account set` and resumes a session there.

Rationale: the board says a test of the pick and headroom needs a real second account the operator provides, and that the task states how it is wired.

### Added after Discussion (arbiter, 2026-11-18): Round-3 hard-cap ruling: F26 to F28 are answered by D7 as fixed

Decision: The three open design-wrong findings of Discussion review round 3 stand answered by D7's text as the round-3 fixer left it, and the review segment is accepted unconverged on round 3. F26: D7 step 4 now runs the relaunch's startup wait with the events proof off, deciding on the pane capture alone, with a criterion for a trust dialog and a pane that never reaches ready. F27: step 4 writes the session id, PromptLine, cleared started flag and prompt offset under the record's lock before reed relaunches the pane, and the bound covers a live strand by clearing the mark and attaching, with DeliverPendingPrompt typing an unsent PromptLine. F28: one running-run predicate on the record covers every reader that compares an outcome to runOutcomeRunning, and Runner.StopStepRuns stops a relaunching run whatever its pane state, with a criterion. Plan and implementation carry D7 as written.

Rationale: Checked against the code: checkLivenessTick's doc (internal/shuttleengine/wait.go) says only the launch's own startup wait sets eventsProof and Wait's probe of a not-yet-started run leaves it unset, so the relaunch's wait matches an existing path; StopStepRuns (stepstop.go) filters on rs.Outcome != runOutcomeRunning and strandRemovable, which D7 now overrides for a relaunching run; DeliverPendingPrompt exists (run.go). Each finding's prescribed fix is in the record, so no fix remains to be made and the hard cap allows accepting.

## Constraints

- `PATTERN-shuttle-provider-seam`: `CLAUDE_CONFIG_DIR`, the credentials file, the status line input, the account file, the pick and the copy live in `internal/shuttleengine/claudeengine`;
  shuttle reaches the switch through an engine-neutral optional capability, a CLI verb reaches the engine only through `internal/shuttleengines`,
  and the in-run relaunch stays inside `internal/shuttleengine`, ending the session's process only through the stop verb's relaunch mode (PATTERN changes)
  and relaunching its pane under the same GUID through reed.
- `PATTERN-config-layers`: the new keys are classified `CategoryMachine` in the module's `ConfigSpec` in the same change; they are new, so no reinterpretation.
- `PATTERN-lym-files`: the account file and the usage directory are named in `internal/lymdirs` under the Bolt's `.lym`, as caches a fresh machine starts without; lock paths come from `internal/lock`.
- `PATTERN-tolerant-records`: the account file, the usage files and the run directory's launched-account record are read tolerantly and rewritten through `internal/state`.
- `PATTERN-cwd-resolution`: the engine gets the account file path, the worktree's name and kind, and for a pick a reader of the hub's current worktree names, as told data, never derives hub geometry.
- `PATTERN-cli-cobra`: `lym claude account` and the status line verb are a cobra subtree under the one root, wired through `internal/cliwire`.
- `PATTERN-engine-parity`: the status line is an engine settings item, listed and never compared, so Codex owes no counterpart.

## PATTERN changes

- `PATTERN-shuttle-provider-seam` — changes: the Shuttle stop section admits the stop verb's relaunch mode (D7): it marks the run record relaunching and keeps the strand; until the mark clears or its bound passes, every running-run reader reads it as live, a step stop stops it, and the orphan sweep keeps it.

## Auto-mode assumptions

- A beams-pinned worktree switches on a limit while its pin is out, and returns to the pin after; a hub-pinned one waits, following the board's "this hub only on private account 1" (D4, D7).
- The beams pin covers every session of a beam worktree, reading the board's spreading workers as the task worktrees under the beams (D1, D9).
- The pick runs only at a worktree's making and never for the prime (D3, D9).
- The lym status line replaces the operator's own in lym panes (D5).

## Open risks

- The manual check (D10) is the first run of a copied transcript resuming under the second account (D6).
- The headroom view is per hub, and several hubs share the accounts; a hub can pick an account another hub has spent until its own sessions see the figures or hit the limit.
- Claude Code may rename the status line fields; the verb then writes nothing and the pick falls back to weights and limit hits.

## Acceptance criteria

- D1: a machine-layer `claude.accounts` list loads; a duplicate name, a relative path, two entries naming one directory once links are resolved, a negative weight, an unknown window or a threshold outside 1 to 100 is refused.
- D1: a non-empty `accounts`, `account_pins.hub` or `account_pins.beams` in a tracked `claude.yaml` is refused, also when the machine file holds the key, naming `lym config claude --layer machine`; a pin naming no listed account, or a retired one, is refused.
- D1: with one account of positive weight no pick runs at worktree making; a pin naming it is still exported.
- D2: an entry whose account name left the list, or whose account is retired, reads as no entry; a missing or empty account file leaves every launch of an unpinned worktree unchanged from today.
- D2: a pick drops the entry of a worktree whose name the told names reader lacks, and keeps the entry of a worktree made while the pick waited for the lock.
- D3: with weights 2 and 1 and both eligible, six picks give four and two, never the second account twice in a row.
- D3: an account with a listed window at or above its threshold and a future reset is passed over; with its reset passed it is eligible.
- D3: an account listing no windows with a future `blocked_until` is passed over.
- D3: an account whose directory lacks `.credentials.json`, or of weight 0, is never picked; with no candidate the name's entry is removed.
- D3: with every candidate short, the pick takes the one whose latest blocking reset is earliest, a tie to the earlier entry.
- D3: an added or reweighted account's count is set to the lowest count over weight among the unchanged accounts of positive weight times its weight; an account listed throughout under one weight keeps its count.
- D3: two picks at once for one worktree end with one entry and one increment.
- D3: the task run's Worktree-Create step, a new beam's creation and an adopt adding a beam or task worktree each pick, replacing an entry the name already had; a pinned worktree and the prime are not picked.
- D4: a worktree whose account is on a non-default directory gets launch and resume commands, and `OpenLaunch` lines, exporting `CLAUDE_CONFIG_DIR` as it after any `env` module export.
- D4: an account on `~/.claude` gets `CLAUDE_CONFIG_DIR` unset; no account gets neither.
- D4: a hub pin wins over the entry and over a beams pin; a beams pin applies to the prime and a beam worktree and not to a task worktree.
- D4: a beams-pinned account with a future `blocked_until` yields to the worktree's entry; a beams-pinned worktree with no entry still launches under the pin; a hub-pinned account never yields.
- D4: the run directory records the launch's effective directory and the listed account it equals, also for a worktree with no entry on a listed directory.
- D4: a tmux server spawned from a process carrying `CLAUDE_CONFIG_DIR` carries none.
- D5: the verb, given a listed name and input with both windows, writes both; given input without `rate_limits`, or an unlisted name, it writes nothing.
- D5: a window outside 0 to 100, or reset beyond its length ahead, is dropped and the other written; the merge keeps the later reset, the higher percentage at one reset, and `blocked_until`.
- D5: a session whose launched account is listed gets settings carrying the status line command with its name, an `OpenLaunch` session with no creel flag directory included; a session on an unlisted directory gets none.
- D5: a usage-limit hit records `blocked_until` on the launched account at its reset, or 30 minutes on with none, and the account has no headroom until then.
- D6: a resume whose transcript sits only under another known directory copies it, with its session directory, into the resolved directory before the resume check stats it, and passes; the source stays.
- D6: a resume whose resolved directory holds an older copy than another known directory gets the newer copy; a transcript found nowhere is refused as today.
- D6: the resume check refuses a session registered live in any known directory's registry.
- D6: an orch resume in a prime whose account changed resumes its session under the new account and does not launch fresh.
- D6: a shuttle resume whose check copied the transcript and whose first turn ends on an API error other than a usage limit starts the session fresh in the same run, and records no API-error re-send for it;
  the orch watcher starts the orch fresh in the same case.
- D6: a shuttle resume and an orch resume through `StartSession` whose check copied the transcript record the copy in the new run's directory,
  also when the first resume was refused for a live holder, and a first-turn API error other than a usage limit then starts the session fresh.
- D7: a shuttle session hitting a limit with another eligible candidate switches the worktree's entry, increments its count, and relaunches inside the same run as a resume of the same session id under the new account; the run records no stopped outcome.
  After the relaunch the launched-account record, the settings' status line name and reed's stored resume command all name the new account, the resume is not refused for a live holder, and the run record keeps its strand GUID.
  The session's first new turn is the continue line, neither a skill load nor the task prompt, and the relaunched pane passed its startup gates.
- D7: an attach probe during the relaunch reads the run live; a relaunch whose `Prepare` fails ends the run as a strand that died.
- D7: an attach after the `Wait` process is killed between step 1 and reed's relaunch in step 4 ends the run as a strand that died once the bound passes;
  a sibling `Start`'s orphan sweep during a relaunch keeps the run directory; a `StopStrand` during a relaunch leaves the run stopped and no strand up.
- D7: an attach after the `Wait` process is killed between reed's relaunch and the mark's clear, once the bound passes, attaches the live run,
  reads the new session id and prompt offset from the record, and does not read the limited turn end again.
- D7: a relaunched pane that shows a trust dialog has it dismissed before the relaunch completes, though the events file already holds the limited session's session-start and turn-start lines;
  a relaunched pane that never reaches ready ends the run as a strand that died, or as a timeout when the run's own deadline passes first.
- D7: a `StopStepRuns` during a relaunch leaves the run stopped and no strand up; a sibling `Start`'s carry-over read does not treat a relaunching run within its bound as unfinished;
  the activity read of a relaunching run reports no usage limit.
- D7: `StopStrand` and attach given the strand's GUID from before the relaunch reach the relaunched run; the stop records stopped and removes the strand.
- D7: a relaunch that falls back to a fresh launch leaves the run record and reed's strand naming the new session id.
- D7: a switched run on a spec with `APIErrorRetries` records no API-error re-send for the limited turn end and does not end `OutcomeAPIError`.
- D7: with no other eligible account the entry is unchanged and the limit is handled as today.
- D7: a beams-pinned worktree whose pinned account hits a limit switches to the entry, and its first launch after the `blocked_until` returns to the pin.
- D7: a hub-pinned worktree whose account hits a limit keeps its pin and switches nothing.
- D7: a worktree with no entry on a listed directory that hits a limit switches to another account, never to its own.
- D7: two sessions of one worktree hitting the limit at once make one switch, and both resume under the new account.
- D7: the orch watcher on a switch ends the still-live limited orch process, then restarts the orch through its resume under the new account instead of waiting;
  after the restart reed holds no old orch strand, the orch state names the new strand, and the new process is registered as the worktree's orch.
- D7: `lym claude account set` writes the entry, refuses an unlisted or retired account, and the worktree's next launch uses it; under a hub pin it is refused, naming the pin.
- D8: `LocateSessions` finds a worktree's sessions under every known directory, a retired account's included, and returns a session copied by D6 once, the most recent copy.
- D8: after `lym claude account set` moves a worktree, a running session's fork audit and outstanding-forks read still read its transcript under its launched directory.
- D8: the liveness read, built for a hub, finds a session registered only under a listed non-default directory; dead needs a dead entry and no unreadable registry.
- D8: `tools/tokencount` and the session-end usage record count a session run on a listed non-default account.
- D9: an orch and a dobby in the prime under a beams pin run under the pinned account; `lym adopt root` writes no entry for the prime.
- Review: the task's landing notes state how the second account is wired (D10) and the manual check's steps.

## Notes for the plan writer

- The names `lym claude account`, `accounts.json` and the launched-account record in the run directory are this record's; Plan may rename them within the same shape.
- No in-run relaunch exists in shuttle today; the stop verb and the API-error retry in `internal/shuttleengine/apierrorretry.go` are the nearest existing paths for D7's relaunch.
- Reed has no op today that ends a strand's process while keeping it, or relaunches a pane under the same GUID with a new command; `ReplaceStrand` mints a new GUID.
- The readers D7's one running predicate reaches are the sites in `internal/shuttleengine` that compare a run record's outcome to `runOutcomeRunning`.
