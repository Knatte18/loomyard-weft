```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals the fixer carried out (F1, F4–F7, F9–F15).

Proposals I challenged:
- F1: holds. Setting `systemMessage` shows each nudge in the Codex pane as `↳ Hook · …`. Claude's nudge sends `additionalContext` alone. But the creel spike only proves delivery with both fields set.
- F4: produces a breakage (finding 1).
- F5: holds.
- F6: leaves a case open (finding 2).
- F7: holds. D6 and D9 each have their own criterion.
- F9: the cited default may be stale (finding 3).
- F10: holds. A refusal fits D4's stance of refusing rather than launching with less.
- F11 through F15: hold.

1. **MEDIUM. D1's new scan, and its twin in the first acceptance criterion: "a scan fails on a production launch-spec literal in a package the table has no entry for".**
   This scan breaks on specs the record means to leave out of parity.
   - The orch spec is pinned to Claude by D4 and is out of Scope. It is still a production spec literal, so the scan fails on the orch packages unless they get an exemption, and D1 names none. D2's scan does skip the orch places explicitly.
   - `internal/shuttlecli`'s hand-written spec (D4) cannot join the producer table: its prompt has no prompt engine to render "for that engine", so it can't take an entry either.
   - Fix: the scan needs the same exemptions D2 states (the orch places and the hand-written CLI spec). Without them, the change fails on code the record keeps.

2. **MEDIUM. D10's new sentence: "with the parity test's hook runner, which lists each plugin hook unrun".**
   D1 says an omitted plugin context "counts as missing in the record's verdict". D10 also prints "the parity verdict" and exits non-zero on a missing item.
   - The record never says whether an unrun context counts as delivered, omitted or something else.
   - Concrete case: on a machine where the scribe plugin's SessionStart hook is on, the verb either always fails (unrun counted as omitted) or the record has a third state D1 never defines.
   - The verdict should state that an unrun context is compared by plugin and hook key only.

3. **LOW. D8's bound: "else Claude Code's 60-second default (Claude Code hooks documentation)".**
   The default is cited from documentation with no version, the same defect F14 fixed for D9. I recall Claude Code's changelog raising the hook timeout to 10 minutes, though I could not confirm it in the installed 2.1.325 binary.
   - If 10 minutes is right, a hanging plugin hook stalls every Codex launch for 10 minutes. That is the stall F9's issue named, now with a wrong number stated.
   - Fix: name the version, or give lym its own bound.

Further tool calls:
- `claude --version` plus a strings search of its binary: to check Claude Code's default hook timeout, which my context lacked. The search failed on a regex limit.
- A second strings search for `TOOL_HOOK_EXECUTION_TIMEOUT`: same reason. It found nothing.
