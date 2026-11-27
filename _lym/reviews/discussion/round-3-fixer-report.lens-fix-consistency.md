```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checking that the round-3 change set agrees with its sibling decisions, criteria and cross-references.

Findings:

1. MEDIUM. The total wait in D10 no longer covers every wait.
   - Changed text: D3 now has every script run wait for the script lock "for at most 30 seconds". D10's step "each script run taking D3's script lock inside it" puts that wait under `kwinplace.lock`.
   - Problem: the twin sentence in D10 was left as it was. It still says "a child that returns to the parent wait waits at most 10 seconds for its window, 180 for its parent and 120 for the lock in all".
   - Evidence: each query or apply run under `kwinplace.lock` can now wait up to 30 s more for the script lock, and the 120-second timer covers only `kwinplace.lock`. So the "in all" bound the sentence states no longer holds.

2. LOW. D10's bound on what the hidden verb reads is stale.
   - Changed text: D3 has each run read and write `kwinscript.loaded` and take `kwinscript.lock`.
   - Problem: D10's bound still says the hidden verb "reads no input but its arguments, the places file and KWin". `kwinscript.loaded` is a new input. The review's Rejected list already flagged this bound as incomplete, and this fix makes it more so.

3. LOW. D3 step 3 reads against D2's rationale.
   - Changed text: D3 step 3 adds "since two windows of one program can share a pid, class and caption".
   - Problem: D2's rationale, unchanged, says "the three together name one window". The new clause is about other programs' windows, but nothing scopes it that way. A reader sees the record claim both that the three-part match names one window and that two windows can share all three parts.

4. LOW. The Constraints API list still names a window's `tile`.
   - Changed text: the same bullet gained `internalId`.
   - Problem: after D3 step 1 and D6 were changed, the design never reads `tile` ("never the window's `tile`"). Its listing among the API facts the design rests on is left stale.

Further tool calls: none.
