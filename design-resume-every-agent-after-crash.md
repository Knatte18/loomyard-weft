# A run agent killed by a crash or reboot resumes its own session instead of starting fresh

`resume-every-agent-after-crash` · Task · enhancement, shuttle, shed · landing · Finalize

## Seen

On 2026-11-23 Neve froze and was reset while `sandbox-without-new-kernel` was 22 minutes into Darn-Write.
Resuming the run spawns a fresh Darn-Write agent: the files it wrote stay in the worktree, but everything it learned in its session is lost.
Only some sessions resume their old session today, through `ResumeSessionID`: PR-Review (`internal/landingshed/prsession.go`), the sampler (`internal/loomshed/samplersession.go`), the orch (`internal/orchcli/start.go`), and any session lym relaunches on an account switch (`internal/shuttleengine/accountrelaunch.go`).
The Claude and Codex engines both take a resume ID, so the mechanism exists for every role.

## Idea

When a row's agent session ended without finishing, by a crash, reboot or killed pane, re-calling that row resumes the agent's recorded session instead of spawning a fresh one, for every role and engine.
A fresh agent stays for a deliberate restart: a goto, a new review round, a re-call after an escalation's answer, or a session whose transcript is missing.
