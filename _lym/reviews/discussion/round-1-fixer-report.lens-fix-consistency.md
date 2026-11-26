```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling decisions, criteria and cross-references they touch.

1. **MEDIUM: the D8 criterion contradicts D3's widened inside test.**
   - Changed text: the D8 acceptance criterion says a daemon refuses every verb but `deliver` and `confirm` "from a command run by an area hub's attach client".
   - Problem: the fix also widened D3's inside test, which now says a process is inside when "its ancestry reaches the recorded supervisor or an attach client the record lists". So the attach client's command is inside its own hub's area.
   - Result: D8's rule refuses only a peer "inside another hub's area" or one "not inside the daemon's own hub's area". The hub's own daemon therefore serves that command under the in-hub rules, while the criterion says it refuses.
   - Needed: the criterion should name "another hub's daemon", or D3/D8 should say which side holds.

2. **LOW: D3's list of the area state directory's contents is now incomplete.**
   - Changed text: the first bullet of D3 says the directory "holds the supervisor record and the supervisor's socket, and nothing the area writes".
   - Problem: the fix's D12 also puts the relay's setup there ("lives in the area state directory, written by the operator's host-side `lym area slack-setup`"). D13 already keeps the deploy clone there.
   - Result: the list reads as complete but is not. This matters because D6 now grants read on "every hub's area state directory", which makes the relay setup and the deploy clone readable from every area.

3. **LOW: D12 leaves the in-area `lym slack setup` without a stated outcome.**
   - Changed text: D12 moves the setup into the area state directory, which is read-only inside, and adds a host-side `lym area slack-setup`.
   - Problem: nothing says what `lym slack setup` does when run inside an area. D12 states the in-area routing for `EnsureRelay`, the away variant and stop, but not for setup. From the file's mode alone, setup would fail on the write with no way forward named.

4. **LOW: the creel socket is placed "under" the area root in D6 but must sit "directly in" a root in D5 and D15.**
   - Changed text: D6 says "The hub's creel socket and its tmux socket live under the area root". The new D5 shape check admits a path only when it "sits directly in a lym temp root or an area root", and the D15 pattern says "`creel-*.sock` directly in a lym temp root or an area root".
   - Result: the two placements agree only through `PATTERN-lym-files`, whose creel socket sits directly in the root. D6 does not say that. If the implementer reads "under" loosely, D5 would refuse the area hub's own socket on other hubs' allow lists.

No further tool calls made.
