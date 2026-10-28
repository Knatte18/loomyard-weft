# Card 18 — Real Claude sessions prove the hook delivery end to end

**Create:**
- `plan:internal/creelcli#TestSmokeCreelHookDelivery` -> `func TestSmokeCreelHookDelivery(t *testing.T)`
- `plan:internal/creelcli#smokeHub.startClaudeRun` -> `func (s *smokeHub) startClaudeRun(t *testing.T, spec shuttleengine.Spec) (name, guid, eventsPath string)`

**Uses:**
- `internal/creelcli#newSmokeHub`
- `internal/creelcli#smokeHub`
- `internal/creelcli#newHubDelivery`
- `internal/shuttleengine#Launch`

**Intent:**
The acceptance criteria that need a real Claude session run in the LLM tier, through shuttle's own rendered `settings.json` and launch line, never a hand-launched session; claim, count, waiter, bell and filter logic are pinned at lower tiers by cards 4 to 15 (`PATTERN-test-speed`).

`TestSmokeCreelHookDelivery` is a `tier3` test in creelcli, beside `TestSmokeCreelClaudeStrand`, run by hand only.
It builds a smoke hub, starts the creel daemon over the hub delivery, and for each scenario launches a Claude session with `startClaudeRun`, which builds a shuttle runner over the hub's reed engine with claudeengine told the hub's config directories, so its settings carry the creel hooks and its strand the `CreelHooks` mark, starts the spec and returns the strand's name, guid and events path.
The test process posts as EXTERNAL.
Each scenario is a subtest, polling the pane and the events file on a tick under a real-time deadline, as the existing smoke tests do:
mid-turn: a prompt that runs several shell commands with pauses gets a post after its first command; the agent acts on it in that turn (it writes the file the post names), and the pane shows the PostToolUse block's system message with the message text;
turn end: a post made after the turn's last tool call is delivered by the Stop block, the turn goes on (the agent writes the file the post names), and `events.jsonl` holds no stamp line or Stop record for the blocked stop and exactly one of each, with the full payload, for the real end;
idle wake: a session idle at its prompt with a draft typed into its box gets a post; within seconds the strand is woken, the draft is still in the box afterwards, the message shows in the pane as the Stop block, and the bell line is never typed into the pane;
successive wakes: three posts, each after the session went idle again, each wake and each deliver through the Stop block;
fork: a fork-mode session asked to run tool calls inside a fork while a post arrives takes the mail only in its main thread: the fork's transcript shows no `[creel]` line, and the main thread's next boundary shows it;
interrupt and bell: `lym creel post --interrupt` to a session in a long turn stops the turn; a user interrupt fires no Stop, so no waiter reaches the stopped session, and the pane then shows the bell line typed exactly once, after which the message shows in the pane as the Stop block of the turn the bell started, and no second bell is typed for that set;
in none of them is a reminder or notice line typed into the pane.
A failing scenario logs its pane and keeps its events file, as the turn smoke tests do.
