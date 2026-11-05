# Codex runs get creel mail through hooks, as Claude runs do

Every Codex strand was a pane-mail strand: `codexengine` installed no creel hooks, so mail from another agent reached a Codex session only as a typed `run lym creel read` reminder.
A Codex run in a hub now carries creel's hook delivery, built on what the hook spike measured on Codex CLI 0.164.0 (`docs/research/codex-hook-delivery-2026-10-28/report.md`), and its launch is marked `CreelHooks`.
The daemon then treats it as a hook strand, the same as a Claude run.

## What a Codex run in a hub carries

- **Tool-boundary claim.**
  A second `PostToolUse` entry, after the activity hook, forms the strand's flag name from `$LYM_STRAND_NAME`.
  Only when that flag stands in the hub's creel flag directory does it run `lym creel hook-read codex`; with no flag it prints `{}` and starts no process.
- **Turn-end delivery.**
  The Stop recording becomes one merged command.
  When the flag stands, it pipes the payload to `lym creel hook-read codex` and exits 0 if mail was handed out, recording no turn end, since the block continues the turn.
  Otherwise it appends the Stop record as before, so `ParseEvents` never reads a Stop that handed out mail as a turn end.
- **Hook output.**
  `HookOutput` renders mail as `{"decision":"block","reason":<mail>}`, the shape the spike saw Codex take at the next step after a tool call and at once after a turn end.
  With no mail it renders `{}`.
- **Idle wake.**
  The Stop setting also carries an async second entry with a timeout of 86400 s: `lym creel hook-wait codex`, with `CODEX_HOME` and `LYM_CODEX_BINARY` set on its command.
  When the daemon rings that waiter, hook-wait calls the new `shuttleengine.IdleWaker` capability.
  Codex's `WakeIdle` runs `codex queue --thread <session_id> --message <mail>` with the session id from the Stop payload, before creel marks the mail read.
  A failed queue leaves the mail unread, and the daemon's bell takes over once the ring goes unanswered.
- **Trust sentence.**
  The prompt file opens with the creel trust sentence, after the deny notice.
  Without it, the spike showed Codex reading a block as a failed tool call and not acting on it.

A Codex engine told no board directory (a standalone run) installs none of this and keeps the plain Stop recording.

## Shared pieces

The creel trust sentence, the flag directory (`CreelFlagDir`) and the shell fragment that turns a strand name into its flag file name move from `claudeengine` into `internal/shuttleengine/creelhooks.go`.
Both engines now use them, and Claude's behavior is unchanged.
`shuttleengine.IdleWaker` is a new optional capability beside `ToolBoundaryContextRenderer`, for a provider whose idle session no hook exit wakes.
hook-wait asks such an engine to start the woken turn, and keeps writing `RewakeOutput` to stderr for every other engine.

## Design decisions

- **The `codex queue` call runs in the waiter process the daemon rings, not in the daemon itself.**
  The daemon still decides when to wake, through its existing ring.
  The waiter already holds what the call needs: the Codex session id from the Stop payload and the session's Codex home.
  It also sits inside creel's claim and acknowledge sequence, so a failed queue leaves the mail unread with no new daemon path, Delivery method or session-id lookup.
  The entry asked for the wake "from the daemon"; this meets the same need with the ring, claim and echo machinery unchanged.
- **The echo of mail an idle wake handed over is dropped for Codex.**
  The operator already sees the queued line as a user message.
  Codex also shows a hook's system message beside the reason, so an echo there could read as new mail.
- **An opened Codex session (`lym shuttle open`) stays a pane-mail strand.**
  It has no prompt file to carry the trust sentence, and without the sentence Codex reads a block as a failed tool call.
- **The hook commands call `lym creel hook-read codex` / `hook-wait codex` from the session's working directory**, which is the anchor Codex runs its hooks in.
  They do not change directory first, as Claude's hooks do with `$CLAUDE_PROJECT_DIR`.

## Tests

- `codexengine`: `TestHookInput` and `TestHookOutput` (tier0) pin the payload parse and the block output, including the dropped echo.
  `TestHookSettings` gains a creel row and pins the waiter entry literally.
  `TestPrepare_CreelHooksInAHub` pins the `CreelHooks` mark and the creel settings for an engine told a board, and the existing Prepare tests pin the trust sentence in the prompt file.
- `codexengine`, tier1: `TestCreelHookCommands` runs the mail hook and the merged Stop hook under `sh`, with a stand-in `lym`, in each flag and outcome case.
  `TestWakeIdle` runs the idle wake against a stand-in `codex` and checks the queue arguments, `CODEX_HOME`, a payload with no session id and a failing queue.
- `creelcli`: `TestWakeSession` pins hook-wait's two wake paths and that a failed wake is returned, so the mail stays unread.
- `shuttleengines`: `TestToolBoundaryRenderer` pins that the `codex` entry renders hook output and is an `IdleWaker`.

## Open risks

Every Codex behavior here rests on one spike sample per case.
These are not yet observed on real sessions, and the live trial task `codex-creel-trial` exists to check them:

- whether an async Codex hook lives through a long idle spell;
- whether `codex queue` works from inside a hook process;
- whether Codex runs an event's hooks in parallel;
- whether a Stop payload carries `stop_hook_active`.

A block after a tool call makes Codex read that call as failed, so a session may run the call once more.
