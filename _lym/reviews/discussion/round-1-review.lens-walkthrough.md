```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Walkthrough list**

- **D1** (purpose met): the operator lists A (`~/.claude`, weight 2, `five_hour: 90`, `seven_day: 95`) and B (`~/.claude-b`, weight 1, `seven_day: 95`) in `claude.machine.yaml`. The machine layer stacks and the engine loads both entries.
- **D2** (purpose missed, findings 1 and 2): the prime worktree right after landing has no record and a pre-change orch session in `~/.claude`.
  - If the dobby's fresh launch runs before the orch's resume, the orch's transcript is lost to it.
  - A hub that sets `CLAUDE_CONFIG_DIR` through the `env` module has its resume broken.
- **D2 with D3, empty list** (contradiction, finding 3): a fresh launch with `claude.accounts: []` and no record goes into D3's pick and fails the launch.
- **D3** (met for the counter, fallback ambiguous, finding 4):
  - With weights 2 and 1, picks go A, B, A, A, B, A.
  - Seeding B against `{A:4}` starts B at 2, and the tie goes to A.
  - In the all-short fallback, A blocks `five_hour` (resets in 1h) and `seven_day` (resets in 3d); B blocks `seven_day` (resets in 2d). "All reset soonest" picks B under a max reading and A under a min reading.
- **D4** (purpose met): a record on B gives env-module exports, then `export CLAUDE_CONFIG_DIR=~/.claude-b`, then the launch. A record on `~/.claude` gives an unset, and a reed relaunch replays the stored command.
- **D5** (purpose met): a session on A carries `statusLine` → `lym &lt;verb&gt; a`. Once the first API response arrives, `rate_limits` is merged into `&lt;Bolt&gt;/.lym/claude/usage/a.json`. D3's next pick reads it per window against A's thresholds.
- **D6** (purpose missed, finding 5): a worktree's record is on B, then B is removed from the list. The record is kept (D2), but B leaves D6's known-directory set.
- **D7** (purpose missed): same case as D2 finding 1. The prime's and beams' pre-change sessions resume only if a resume precedes every fresh launch there.
- **D8** (purpose met): a session on B hits its limit. The orch watcher waits, and the resume command carries B's directory from the record.
- **D9** (purpose met): the operator links `&lt;dir&gt;/plugins` to `~/.claude/plugins` and lists both accounts. A task run's record, transcript and usage file land as the check expects.

**Findings**

1. BLOCKING · design · D2 "How a session launch reads it", D7
   - **Summary:** whether a worktree's pre-change sessions survive depends on launch order, and the first fresh launch strands them.
   - **Steps:**
     1. Prime worktree after landing: no record, orch transcript in `~/.claude/projects/`.
     2. The dobby, or any lym session, launches fresh first. D2's fresh rule picks B and writes the record.
     3. The orch's resume reads a record whose directory exists, so it keeps the record ("kept, whether or not…").
     4. D4 exports B's directory. `CheckResume` stats `&lt;B&gt;/projects/&lt;enc&gt;/&lt;id&gt;.jsonl`, which does not exist, and refuses.
   - **Purpose missed:** D2 ("resume their own") and D7 ("sessions started before this change resume and record the default directory") both fail. Task worktrees mid-run at landing break the same way, when a fresh reviewer spawns before the writer's resume.
   - **Evidence:** `decision-record.md:66-73` and `:179`; `internal/shuttleengine/claudeengine/resume.go:29-38` (registry and project dir read from the home `.claude`), `:54-58` (no transcript refuses).
   - **Gap:** no decision has a fresh launch with no record look for existing transcripts of the worktree in the known directories before it picks.

2. BLOCKING · design · D2 resume search set against D6's known directories and D4's unset
   - **Summary:** with the feature off, a hub using `env`-module `CLAUDE_CONFIG_DIR` has every resume broken.
   - **Steps:**
     1. Today a hub sets `CLAUDE_CONFIG_DIR=/x` through `lym config env` (board "Built since"); `claude.accounts` is empty.
     2. The orch resumes with no record. D2 searches "the listed accounts' directories and the default directory" (none, plus `~/.claude`). The transcript is in `/x`, so it is found nowhere and the record says `~/.claude`.
     3. D4 sees the default as the recorded directory and unsets `CLAUDE_CONFIG_DIR` after the env exports (`panebin.go:67-80` exports `PaneEnv` ahead of the launch command).
     4. Claude runs under `~/.claude` and the resume check refuses.
   - **Contradictions:**
     - D2's search set is narrower than D6's known set (`decision-record.md:165`) even though it cites D6.
     - It breaks D4's "With no record and no accounts, the engine exports nothing and an `env` module value stands as today" (`:115`) and the D1 criterion at `:231`.
     - It breaks Scope's promise to keep today's behaviour, since this hub never opted in.
   - **Evidence:** `decision-record.md:70-73`, `:114-116`, `:165`.

3. MEDIUM · consistency · D2 fresh-launch rule with D3 step 2, against D4 and D1's acceptance criterion
   - **Summary:** an empty accounts list fails every fresh launch as the record is written.
   - **Steps:** D2: "A fresh launch with no record … picks an account (D3)". With `claude.accounts: []`, step 2 has no candidate, and "the session launch fails". That contradicts D4's "With no record and no accounts, the engine exports nothing" and the D1 criterion that such a launch "neither sets nor unsets".
   - **Gap:** no decision says the pick and the record write are skipped when the list is empty.
   - **Evidence:** `decision-record.md:68`, `:93`, `:115`, `:231`.

4. LOW · consistency · D3 step 4
   - **Summary:** "the candidate whose blocking windows all reset soonest" gives different picks for a candidate with several blocking windows.
   - **Steps:** A has blocking windows resetting in 1h and 3d; B has one resetting in 2d. The max reading (A free in 3d) picks B. The min reading (A's first reset in 1h) picks A, which is still blocked after that hour.
   - **Gap:** no tie-break is stated, unlike step 3.
   - **Evidence:** `decision-record.md:96`, criterion at `:239`.

5. MEDIUM · design · D6 known-directory set against D2's kept record
   - **Summary:** a worktree kept on a delisted account drops out of liveness, report and tokencount reads.
   - **Steps:**
     1. A worktree is recorded on B, and B is removed from `claude.accounts`. D2 keeps the record, and its sessions keep running under B.
     2. D6's known directories are the listed accounts, the `env` module value, the process value and the default, so B is not among them.
     3. The id-only liveness read (`prober.go:31-35`, per D6 extended to every known directory), `lym report` and `tools/tokencount` all miss B's sessions.
   - **Purpose missed:** D6's rationale ("a reader of `~/.claude` alone loses every session on that account").
   - **Gap:** the worktrees' recorded directories are not part of the known set.
   - **Evidence:** `decision-record.md:66-67`, `:162-165`, `:171`.

Tool calls: none.
