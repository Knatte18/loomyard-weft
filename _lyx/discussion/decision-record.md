# Decision record — sed-read-ok

## Goal

Read-only `sed` works for every lyx-spawned agent with no refusal, prompt or false audit halt, while rewriting a file with a script stays banned and becomes enforced.
A Webster fork, or any agent, can no longer stall a run on a Claude Code permission prompt unseen: the shell forms that raise one are steered away before they run, and a dialog that still appears reaches the operator through a notice.

## Scope

In:

- The shared edit directive's wording: reading with `sed -n` is allowed, the in-place forms are what is banned, and scratch files have a named place, excluded from git in every repo lyx wires.
- Two new standing PreToolUse denies in `claudeengine`'s deny table: an in-place script edit, and `rm` on a computed path.
- The python deny's steer naming `sed -n` among the readers.
- The fabric-reference read-only classifier admitting more `sed` print forms.
- A shuttle notice to the run's parent when an autonomous run's pane shows a permission or input dialog past a bound.
- `PATTERN-no-denied-recovery`'s closed list and its scan following the new denies.
- The docs that describe each of these.

Out:

- A settings allow rule for `sed` (D8); no hook or audit refuses `sed -n` today, and no orch stencil, plugin skill or `CLAUDE.md` carries a "never sed" wording, so only the directive changes (D1).
- The other readers of the classifier (`cat`, `grep` globs and flags) and the `doc.go` reason for globs after `--`: board note `audit-reader-globs` owns them.
- Removing temp files tests leave, and any cleanup command: task `test-tmp-cleanup` owns them.
- A rule about private tmux servers: a fork's `tmux -L <socket>` touches no lyx server and raised no prompt in the incident.

## Decisions

### D1 — The edit directive states the rule by its in-place forms and names a scratch place

`contracts/stencils/edit/edit-directive.md` keeps "change files only with Edit or Write" and the one-Edit-per-site and `replace_all` sentence.
Its script ban names the rewriting forms, each directly after its own "never" so `PATTERN-no-denied-recovery`'s exemption marker admits it: never `sed -i`, never `perl -i`, never `awk`, and never an inline `python` or `node` heredoc.
It states that reading with `sed -n`, `grep` or `cat` is fine.
It adds one sentence: scratch files go under `.scratch/` in the working directory, never a system temp directory, and are removed by their literal path.
`internal/editdirective/doc.go` restates the rule summary to match.
In `editRulePhrases`, "`sed`" becomes "`sed -i`", so the single-statement check still finds the directive and no stencil that mentions reading with `sed`.

Fabric's git-exclude seeding (`internal/fabricengine`'s `gitexclude.go`) adds `.scratch/` to each wired repo's `.git/info/exclude`, so the place is ignored in a target repo that does not list it in its own `.gitignore`.
`.scratch/` is agent scratch, not lyx state, so `PATTERN-durable-vs-ephemeral-state` and `PATTERN-lyxdirs-single-declarer` do not apply.

Rationale: the directive is the rule's only statement and reaches every spawned role's opening stencil, and Webster's implementer forks through Merriam's context, so one edit there covers every agent.
The current "never `sed`" reads as a ban on all `sed`, the stale wording the operator retired from `scribe:conversation`.

### D2 — A standing deny refuses an in-place script edit

A new row in `claudeengine`'s `standingDenies` denies a Bash command that runs, in command position (the existing `commandPositionPattern`), `sed` with an in-place flag anywhere among its words (`-i`, `-i<suffix>`, `--in-place`, `--in-place=<suffix>`, or a short-flag cluster holding `i` such as `-ni` or `-Ei`), or `perl` with `i` in a short-flag cluster (such as `-pi` or `-i.bak`).
Its steer says scripted in-place edits are refused under the edit directive's rule and that reading with `sed -n` is fine; it points at the rule and does not restate it.
It is installed in every run, the orch included, with no config key.

Bound: a guardrail, not a barrier, stated in its row comment and `claudeengine`'s doc.
It lets through `sed` writing by redirection (`sed ... > f`), a `w` command inside a script, `awk`, and anything behind `bash -c`, `eval` or a variable.
It falsely denies a matching spelling in a quoted argument, in a `grep` alternation such as `'x\|sed -i'`, or in another payload field, as the python row does.

Rationale: the directive reached the fork and did not stop its `sed -i`; why the fork ignored it is unknown, and a deny at the moment of the call makes the answer moot.

### D3 — A standing deny refuses `rm` on a computed path

A new row denies a Bash command with `rm` in command position whose command also holds a command substitution (`$(` or a backtick) or a shell variable expansion (`$` followed by a letter, `_` or `{`).
Its steer and notice say to remove a file by its literal path; the scratch place stays D1's alone.
It is installed in every run, with no config key.

Bound: a guardrail, not a barrier.
It falsely denies an `rm` of a literal path in a command that uses `$` elsewhere, a `grep` alternation naming `rm`, and a matching spelling in another payload field;
it lets through a removal behind `find -delete`, `xargs rm` with no `$`, `bash -c`, or a script.

Rationale: Claude Code raises "Dangerous rm operation on statically-unresolvable target" even in bypass mode, so the only way to keep that stall from happening is to refuse the call before Claude Code evaluates it; a deny costs one turn, a prompt costs the run.

### D4 — The python deny's steer names `sed -n`

The python row's steer becomes "edit files with Edit or Write, search and read with grep, awk, cat or sed -n", kept free of quotes and backslashes as its init check requires.

Rationale: it is the one deny steer that lists readers, and it should not imply `sed` is off limits; `awk` stays a reader, since D1 bans it only as an edit.

### D5 — The fabric-reference classifier admits print scripts with regex addresses and several commands

`fabricengine.IsReadOnlyCommand`'s `sed` form stays `sed`, then the quiet flag, then one script word, then file words, and widens to:

- the quiet flag as `-n`, `--quiet` or `--silent`;
- a script of one or more print commands joined by `;`, each an address or an address range followed by `p`, with nothing after the last `p`;
- an address that is a line number or a regex `/RE/` whose `RE` is non-empty and holds no `/`, `;` or newline, with no flag after its closing `/`.

The word-splitting rules stay: no `$` or backslash in any word, so `$p` and an escaped `/` stay non-read-only.
Every other script (`s`, `w`, `r`, `e`, a trailing `;`, an empty regex, a regex flag such as `I`, `-i` anywhere, `-e`, `-s`, `-z`) stays non-read-only.
The function's comment, `websterengine`'s `doc.go` audit section and the `audit: fabric-reference` row of `contracts/specs/refusal-spec.md` state the admitted form; the spec row names no denied form unless "never" precedes it.

Bound: the widening admits only reads; the grammar has no command that writes, and a regex address cannot hide one because `/`, `;` and newline end or split nothing inside it.

Rationale: agents commonly read a function with `sed -n '/^func X/,/^}/p'`; misreading that as a write turns a harmless fabric reference into a correctness halt.

### D6 — Shuttle notifies the parent when the pane shows a dialog lyx cannot answer

In `internal/shuttleengine`'s wait loop, an autonomous run on a runner with a notifier keeps a pending-dialog latch:

- An ask signal that is not an `AskUserQuestion` (the engine's permission and input-dialog signals) sets the latch, stamped with the signal's time and events offset, when it is not already set.
  A further ask while the latch is set changes nothing, so dialogs close together give one notice.
- The latch is independent of the session-state fold: a later turn end or turn start, the main session's or a fork's, does not clear it.
  It clears only when a pane read through the engine's input-box reader shows an input box.
- Once the latch is `permissionNoticeBound` old, a new package constant of 60 seconds, the loop reads the pane on each poll tick until it shows an input box or the notice is sent.
  A read showing no input box sends one notice; a failed read sends nothing and is retried on a later tick.

The notice, built like the held-turn-end notice and with any agent text delimited the same way, names the strand, says its pane shows a permission or input dialog lyx cannot answer, and gives the way forward: the operator answers the dialog in the pane, reached with `lyx reed attach`.
The latch's offset is persisted before the notifier is called, in a run-state field of its own beside `RunState.NotifiedOffset`, so a held-turn-end notice and this notice never suppress each other.
On attach, the loop rebuilds the latch from the events after the persisted offset, so a dialog still pending across a driver restart is still notified once and a sent one is not repeated.
`Spec.QuietHold` and `Spec.HoldDeferral` do not suppress or defer it, since no peer can answer a dialog.
An interactive run, a runner without a notifier, and an engine without session signals or an input-box reader send nothing.
The loop's classification of the run is unchanged, and the run stays bounded by its deadline and liveness check as today.

Bound: the edge sends at most one line per latch episode to the parent that already receives hold notices, never types into the asking session, and never changes an outcome.
`internal/shuttleengine/doc.go` and `wait.go`'s header state the notice in provider-neutral terms; `claudeengine`'s doc names which hook events are its ask signals.

Rationale: today the fold reads a permission ask as `asking` but only logs it, and `ParseEvents` sees no ask, so a stalled run reads `running` until its deadline.
A fork's dialog is raised while Merriam's main turn or its backgrounded wait goes on, so a later main-session signal must not clear it; the pane is the one source that shows whether the dialog is still up, and it also removes a false notice after an operator approved one.
The runners for loom, Webster and burler already route notices to the orch through `orchcli.NotifyPrime`, which queues through `orchengine` as `PATTERN-orch-pane-single-writer` requires.

### D7 — `PATTERN-no-denied-recovery` follows the new denies

`sed -i` and `perl -i` join the closed list of denied forms in `pattern/PATTERN-no-denied-recovery.md` and `cmd/lyx/norecoverydeny_test.go`.
The computed-path `rm` deny is pattern-shaped, so, like the raw `go` deny, it stays outside the closed list and its token scan, and the background file says so; a plain `rm <path>` stays a valid step.
The PATTERN.md one-line entry is unchanged.

Rationale: the entry requires its list to follow `claudeengine`'s deny table.

### D8 — No allow rule for `sed`

lyx adds no `permissions.allow` rule for `sed` to the settings it gives spawned agents.

Rationale: autonomous runs and the orch run with `--dangerously-skip-permissions`, where `sed -n` never prompts, and no lyx hook matches `sed`.
Claude Code's own read-only command check covers `sed -n` prints in a prompting interactive run.
A prefix rule such as `Bash(sed -n:*)` would also admit `sed -n 's/a/b/w f'`, a write, so it would weaken the edit rule.

## Constraints

- `PATTERN-shuttle-provider-seam`: the deny rows, their patterns and the mapping from Claude hook events to ask signals live under `internal/shuttleengine/claudeengine`; D6's latch in `shuttleengine` reads only provider-neutral signals and the engine's input-box reader.
- `PATTERN-orch-pane-single-writer` and `PATTERN-pane-text-injection`: D6's notice goes through the runner's notifier only; nothing types into the asking pane.
- `PATTERN-completion-signal`: D6 finalizes no outcome.
- `PATTERN-daemon-wakeups`: D6 adds no wait of its own; its checks ride the existing poll tick.
- `PATTERN-no-denied-recovery`: changed by D7 as stated; no refusal, stencil, spec or skill names `sed -i`, `perl -i` or an `rm` on a computed path as a step.
- `PATTERN-edit-directive`: D1 edits the one directive file, rendered through `internal/editdirective`; the D2 and D3 steers point at the rule rather than restate it.
- `PATTERN-never-force-add`: D1's `.scratch/` exclusion goes through `.git/info/exclude`.
- The deny steers and notices hold no single quote, double quote or backslash, as `settings.go`'s init check requires.
- `PATTERN-test-speed` and `PATTERN-test-economy`: new cases extend the existing deny, classifier and hold-notice tables where they exist, under an injected clock; the one live check (last acceptance criterion) is llm-tier.

## Auto-mode assumptions

- Claude Code's permission and input dialogs replace the input box in the pane, so the input-box reader reads no box while one shows.
- Claude Code honours a PreToolUse deny before its own "statically-unresolvable target" check, so D3's deny prevents the prompt.
- Claude Code's built-in read-only check admits `sed -n` prints without a prompt in a prompting interactive run.
- 60 seconds is long enough that an operator answering a dialog at once sees no notice, and short enough to surface a stall well inside any run's timeout.

## Open risks

- The live check may show that Claude Code fires neither `PermissionRequest` nor a `permission_prompt` `Notification` for a fork's dialog; that fails at verify, before landing, and means D6's latch needs another trigger.
- A Claude Code release that changes the dialog's pane rendering could make the input-box read report a box while a dialog shows, silencing D6; the shadow fold's state log still records the ask.
- Task `test-tmp-cleanup` plans a sweep of `.scratch/`; it must take only age-stale entries, or it can delete a live run's scratch files.

## Acceptance criteria

- The rendered edit directive allows `sed -n` reading, puts "never" directly before each of `sed -i`, `perl -i`, `awk` and the interpreter heredoc, names `.scratch/` as the scratch place, and the single-statement check finds it alone.
- A repo wired by fabric has `.scratch/` in its `.git/info/exclude`.
- The in-place deny refuses `sed -i 's/a/b/' f`, `sed -i.bak ...`, `sed -ni ...`, `sed --in-place ...`, `sed --in-place=.bak ...`, `sed -n 3p f -i`, `perl -pi -e ...`, a path-prefixed `/usr/bin/sed -i`, and each after `&&`, `;` or `|`; it admits `sed -n 1,5p f`, `sed 's/a/b/' f`, `grep -i x f` and `git diff`.
- The computed-path deny refuses `rm -rf "$(cat x)"`, `rm $DIR/f`, `rm ${D}/f` and an `rm` behind backticks; it admits `rm -f .scratch/a` and `rm a b`.
- Both new denies are announced in the system-prompt notice of every run mode, and the hook and notice tables stay in step.
- The python deny's steer names `sed -n`.
- The classifier admits `sed -n '/^func X/,/^}/p' f`, `sed --quiet 3p f`, `sed -n '1,5p;20,30p' f` and `sed -n '/a/p' f`, and rejects `sed -n '1w out' f`, `sed -n 's/a/b/w f' f`, `sed -n '$p' f`, `sed -n '/a;b/p' f`, `sed -n '1p;' f`, `sed -n '//p' f`, `sed -n '/a/Ip' f`, `sed -i 3p f`, `sed -n 3p -i f` and `sed -n 3p f -i`.
- An autonomous run whose events hold a permission ask (including a fork-shaped payload carrying `agent_id`) and whose pane shows no input box sends exactly one notice once the bound has passed, and none before it.
- A fork's ask followed by a main-session turn end with outstanding tasks, with the pane still showing no input box, still sends the notice.
- No notice is sent when the pane shows an input box before the bound, for an interactive run, or for a runner without a notifier; a `QuietHold` run still sends it.
- Two asks within the bound send one notice.
- A deferred held-turn-end notice pending when a dialog notice is sent is still sent when its deferral ends.
- A run reattached with a dialog pending and unnotified sends the notice once; one reattached after the notice was sent does not repeat it.
- The `PATTERN-no-denied-recovery` scan fails a planted `sed -i` or `perl -i` step and passes the directive's wording.
- A live llm-tier scenario in which a fork runs a command Claude Code prompts on in bypass mode records an ask signal in the run's events file.

## Notes for the plan writer

- The deny table and its notice builder are in `internal/shuttleengine/claudeengine/settings.go`; the python and raw `go` rows are the models for D2 and D3.
- The session fold is in `internal/shuttleengine/sessionstate.go` and is driven from `waitshadow.go`; the hold notice and its once-per-offset persistence are in `hold.go`; the input-box reading is in `readiness.go`.
- The classifier is `readOnlySedSegment` in `internal/fabricengine/refscanner.go`.
- D6 sends through the runner's notifier; board notes `creel` and `orch-split` plan to change who receives notices, so nothing in D6 should name the orch as the receiver.
