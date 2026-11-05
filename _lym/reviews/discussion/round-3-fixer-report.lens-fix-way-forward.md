```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked whether each refusal, next step and acceptance row the fix added can be reached and leads to progress.

Ways forward the fix introduced:
1. D2's `lym slack login` / `lym slack setup` refusal, unknown-pane case included. It names the operator's terminal, plus D6's wait until the pane process is known or the strand ends.
2. D3's `lym slack start` refusal, unknown-pane case included.
3. D6's pane-ancestry refusal when a live strand's pane process is unknown. It names the strand and the wait, beside the operator's terminal.
4. D3: when creel refuses the relay's first read, the relay logs the refusal under `.lym/slack/` and ends, releasing its lock. A later refused read is retried with the poll's backoff.
5. D3 and D2: with the token file or the setup file missing, the relay and `lym slack away` refuse and name `lym slack login` and `lym slack setup`.
6. Acceptance rows D2, D3 (`lym slack relay` from a non-orch pane, then a later start takes the lock) and D6 (the unknown-pane refusal).

Items 1, 2, 3, 5 and 6 can be reached and lead to progress. The D6 wait ends when the pane listing reports the pane's process or the strand stops being live. A missing token file or setup file is fixed from the operator's terminal. A later start takes a lock that a relay ended on a refused first read has released.

Finding:

- **MEDIUM**, D3, the new text "When creel refuses that first read, the relay … ends, releasing its lock".
  - **Summary:** a relay started by the prime bring-up or an anchor change ends when creel refuses its first read. With presence `terminal`, nothing starts it again, so it does not recover by itself.
  - **Evidence:** D4 names a live strand's unknown pane process as a state in which creel refuses a program claim from outside every pane. D3 restarts the relay only at the prime bring-up, an anchor change, `lym slack away`, `lym slack start`, or a prime dobby tick, and a tick restarts it only "while presence is `slack`".
  - **Contradiction:** if a pane process is unknown at an anchor change with presence `terminal`, the relay ends and stays down. That breaks D3's acceptance row "the relay is live after a prime anchor change with presence `terminal`, whether the change lands or fails".
  - **No way forward:** the relay's log holds only creel's refusal ("…cannot be told from a strand; nothing was done"). Nothing tells the operator to run `lym slack start` or tells the bring-up to try again. Before the fix, the record did not say the relay exits in this case.

Tool calls beyond my context:
- `grep PanePID` over `internal/`: to find where a strand's pane process is set and whether an unknown one can become known, so the D6 wait can end.
- `internal/reedengine/strand.go` `safeReapRoot`: to see when a live pane reports no process id, for the same reason.
