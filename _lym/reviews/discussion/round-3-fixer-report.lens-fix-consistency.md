```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the round-3 change set agrees with the sibling decisions, criteria and cross-references it touches.

**1. MEDIUM — D6's new unknown-pane sentence, and D2's refusal bullet that reuses it**
- **Changed text:** D6 now says the unknown-pane refusal names waiting "beside the operator's terminal". D2's `login` and `setup` refusal, which now includes that refusal, still names "the operator's terminal as the way forward".
- **Summary:** In the unknown-pane case, the operator's terminal is itself refused, so naming it as a way forward contradicts the check's own definition.
- **Evidence:** D6 defines the check as refusing "a caller in no known pane while any live strand's pane process is unknown". The operator's terminal is a caller in no known pane. The changed D6 acceptance bullet also says a run "from outside every pane while a live strand's pane process is unknown" refuses. This matches creel's `resolveSender`, which refuses whenever `unreadable != ""` and the peer is in no pane (`internal/creelengine/identity.go:68-70`). So in that state, the only way forward that works is waiting.

**2. LOW — D2's second Bound line**
- **Changed text:** "a process of the operator's user that writes the files directly, or that runs outside every pane, can still repoint them, as D6 and D7 state for presence and creel."
- **Summary:** The cross-reference claims D6 states the direct-write case, but D6 does not. D6's own bound still says no process inside a non-orch strand's pane changes presence. Presence lives in a file in the same `.lym/slack/` directory, so D2 and D6 now disagree about what a pane process writing a file directly can do.
- **Evidence:** D6's Bound reads "no process inside a non-orch strand's pane changes it, but a strand that deliberately starts a process outside its pane is not stopped". It says nothing about writing the presence file directly. D6 also stores presence "under `.lym/slack/` through `internal/state`", the same directory as D2's new setup file.

**3. LOW — D3's acceptance bullet for `lym slack start`**
- **Changed text:** D3's `start` bullet now says "its unknown-pane refusal included".
- **Summary:** The change was added to D2 and D6 acceptance but not to its twin in D3. D3's acceptance still checks only "`lym slack start` run from a non-orch strand's pane refuses and starts nothing", with no unknown-pane case.
- **Evidence:** The D2 and D6 acceptance bullets each gained "or from outside every pane while a live strand's pane process is unknown". The D3 acceptance bullet gained only the `lym slack relay` clause.

**4. LOW — support-log Turn 2 against the new D2**
- **Changed text:** D2's heading and its first bullets now put the ids in a per-hub setup file.
- **Summary:** The support log's Turn 2 still says "token in a credential file, ids in machine config (D2)". It is history, but nothing beside it marks it as superseded, apart from the new Round 3 entry further down.
- **Evidence:** `support-log.md` Turn 2 line 19, compared with the D2 heading "channel and operator in a per-hub file only setup writes".

Further tool calls: none.
