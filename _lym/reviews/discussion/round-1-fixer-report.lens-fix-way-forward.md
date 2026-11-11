```yaml
lens: fix-way-forward
outcome: reported
```

The task: check that every way forward the fix introduced (refusals, next steps, criteria) can be reached from the state it names and leads to progress.

Ways forward the fix introduced or rewrote:
1. D2: a resume refused over a gone directory says to "remove the record so the next fresh session searches and picks again".
2. D2: a changed record reaches a running strand "only when a launch the engine builds starts it again, as a fresh session or a resume".
3. Auto-mode assumptions: "The operator changes it by writing the record with another directory".
4. D3 step 2: on macOS a never-logged-in account's sessions "stop at Claude Code's login prompt until the operator logs it in".
5. D5: the verb's refusal ("writes nothing and prints the line without figures") and its dropped windows.
6. New criteria: D1 empty-list resume, the D2 search rows, the D3 resume+fresh concurrency row, the D4 opened session, the D5 verb rows, the D6 landed-run row.

Findings:

- **MEDIUM.** D2 resume-refusal way forward: "remove the record so the next fresh session searches and picks again".
  - **Summary:** In the prime or a beam, or wherever a resume runs first after the record is removed, no pick happens. The resume searches, finds its transcript nowhere (it was in the gone directory) and records the fallback directory. The worktree is then pinned to the `env`/process/default directory, not picked.
  - **Evidence:** the D2 resume bullet: "A transcript found nowhere is recorded as the directory D6's no-record fallback names … The resume check then refuses as it does today."
  - **Effect:** "picks again" holds only when a fresh launch is the first launch after the removal. Either a resume whose transcript is found nowhere writes no record, or the way forward should say the next launch records the fallback unless it is a fresh one.
- **MEDIUM.** D2 last paragraph: "reaches a running strand only when a launch the engine builds starts it again, as a fresh session or a resume; until then a relaunch keeps the old directory, and the refusal of a gone directory above does not run for it".
  - **Summary:** It states the bound but names no step an operator can carry out to get a relaunched strand onto the changed record, or out of a gone directory.
  - **Evidence:** No verb is named. The review's fix asked for the way forward to be stated ("stop and resume it through shuttle"), and the reed relaunch path replays the stored command (D4 third bullet).
  - **Effect:** A strand whose directory is gone keeps being relaunched into it, with no named exit.
- **LOW.** Auto-mode assumptions: "The operator changes it by writing the record with another directory".
  - **Summary:** The way forward is to hand-write a machine cache that D2 says is "written through `internal/state`" and holds name, directory and time. No verb exists and no format is stated, so an operator has to guess the JSON shape.
  - **Evidence:** D2: "It holds the account name, empty when no listed account names the directory, the config directory and the time it was recorded".
  - **Effect:** It can be done, since the record is read tolerantly (PATTERN-tolerant-records), but the decision should name the minimal hand-written form, a `config_dir` key alone, so the step is reproducible.

Items 4, 5 and 6 are reachable and lead to progress. The macOS login prompt is answered in the pane, as D9's `/login` wiring shows. Each new criterion names a state a test can set up with fake directories.

Further tool calls: none.
