```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every claim the fix added to D2, D3 and D6 against the code at HEAD.

**MEDIUM: three of the named owners never resume their session**
- Changed text: D2's fresh-launch bullet says "the gate fixer's record, webster's state, with the master session and the batch and fixer sessions, and treadle's state each keep a session id their owner resumes after that sweep". The rule sentence above it uses the same "its owner later resumes" test.
- Summary: only some of the named owners resume. The gate fixer, webster and treadle store a session id but never resume that session, so the sentence is false for them, and the rule's "later resumes" test would leave them out anyway.
- Evidence:
  - The only places that set `spec.ResumeSessionID` are `internal/loomshed/samplersession.go:217`, `internal/landingshed/prsession.go:80` and `internal/orchcli/start.go:713`.
  - `internal/gatefixer/spawn.go:126` only records the id.
  - `internal/websterengine/recordbatch.go:440` reads the batch session's transcript; it does not resume it.
  - Treadle's "resume" in `internal/treadleengine/state.go:75-107` is its round loop, not a provider session.
  - The claim comes from the reviewer's F1 fix.

**MEDIUM: reed's strand list is not live-only**
- Changed text: D2's fresh-launch bullet, "reed's strand list holds only live strands".
- Summary: reed keeps non-live strands and relaunches them. It drops one only when all the paths in its done-when list exist.
- Evidence: `internal/reedengine/lifecycle.go:184`: "planResumeLaunches splits the non-live strands into those to relaunch and those to drop". Also `lifecycle.go:802`.

**MEDIUM: "never on any account" is stronger than the guard**
- Changed text: D2's last bullet, "two processes never drive one session on any account".
- Summary: today's check lets a resume through, with only a warning, when the registry directory cannot be read. The guard therefore does not prove "never", even on the default directory.
- Evidence: `internal/shuttleengine/claudeengine/resume.go` `checkResumable`, where an unreadable registry "returns a warning and the check proceeds". The claim comes from the reviewer's F2 fix.

These claims hold:
- The D6 combining rule is new behaviour, and `processLiveness` follows the single-registry rule it cites.
- `readRegistry` errors only when the directory itself cannot be read, a missing directory included. The record now states that the new rule treats a missing directory as no doubt.

Tool calls:
- `grep` of `ResumeSessionID` assignments plus a read of `resume.go`: my context did not show which owners resume, or the registry read code.
- `grep` for resume use in gatefixer, webster and treadle: to test whether those owners resume.
- `grep` of `SessionID` use in those packages: to confirm none of them resumes.
- `grep` of gatefixer's use and reed's strand listing: to test the live-only claim.
