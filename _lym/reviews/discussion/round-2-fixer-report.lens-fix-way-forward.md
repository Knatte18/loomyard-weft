```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that each refusal, next step and test the fix added can be reached and leads to progress.

**1. LOW: the new D4 sentence's way forward names a session the row cannot pick up cleanly.**
- **Changed text:** D4, "otherwise its `decision add` is refused until `lym loom resume` re-enters the row and re-records it".
- **Summary:** suppose a session is reopened outside the row under a strand name other than the recorded one. On `lym loom resume`, D5's ensure finds the recorded strand not live, so it resumes the recorded session id. That id is already live under the other strand. Nothing in D5 or D6 stops that other strand or adopts its name. The row either runs a second copy of the session or never re-records the reopened strand, so the stated way forward makes no clean progress. The new D6 entry bullet has the same blind spot: it stops "the recorded session when its strand is live", so it misses a live session under another name.
- **Evidence:**
  - D5: "a recorded session id whose strand is not live is resumed … each resume or start re-records the strand's name, guid and session id".
  - D6 entry bullet: "first stops the recorded session when its strand is live".
  - `lym reed resume` replays persisted strands under their persisted names (`internal/reedcli/resume.go:19-26`, "for every persisted, non-hidden strand"). So the "otherwise" branch arises only from the stop chain, if at all. The sentence could say a reopen keeps the recorded name instead of promising a recovery the row does not perform.

**2. LOW: the new refusal's way forward disagrees with D4's refusal sentence below it.**
- **Changed text:** D4, the same new sentence.
- **Summary:** the new sentence makes `lym loom resume` the way forward for a refused reopened strand. The next line of D4 says refusals carry "a way forward naming `--by operator` or the run's parent". A reader can't tell which of the two the refusal names.
- **Evidence:** D4, line 77: "Any other label from that strand, and that strand once the row has ended, are refused with a way forward naming `--by operator` or the run's parent."

The other new ways forward hold:
- The D5 `DenyPublish` steer ("commit your change and tell the operator") can be carried out in an interactive pane, and D8 names `sampler-done`.
- In the D6 entry dirty-checkout path ("ensured, resumed when the row stopped it"), D5's ensure resumes or starts a session.
- The new D6 acceptance case can be reached: a session left live while halted, then `lym loom resume`.

Tool calls:
- `grep internal/reedcli` for strand naming on resume: my context did not show how `reed resume` names reopened strands.
- `grep internal/reedcli/resume.go`: confirms that `reed resume` replays persisted strands under their persisted names.
