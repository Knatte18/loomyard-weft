```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether every sibling decision, criterion and cross-reference still agrees with the change set.

1. LOW. **D4 still promises no change for a machine without a file.**
   - Changed text: the Goal's new exception for a relative `HOME` or `LOCALAPPDATA`.
   - Issue: D4's last sentence, "so no machine without a file changes behaviour", still makes the old promise, and the Goal's twin edit now limits it.
   - Evidence: the Goal says a relative `HOME` "now makes its name an error until the variable is set to an absolute path (D3)". D4 still ends "The hostname rule's output is not checked against it, so no machine without a file changes behaviour." The D4 claim is narrower (it is about the name rule), but it reads as the same unqualified promise.

2. LOW. **The collision criterion still says "former name".**
   - Changed text: D5's collision bullet, now "not one of this machine's names", which includes the hostname rule's name.
   - Issue: the acceptance criterion was not updated to match.
   - Evidence: the criterion still reads "`lym machine name neve` is refused when a swept board records `neve` as an owner that is not this machine's former name".

3. LOW. **The decodable-file repair branch still says the file keeps its `names`.**
   - Changed text: D5's write step, which now drops every empty or invalid `names` entry other than the hostname rule's name and adds the hostname rule's name.
   - Issue: the repair branch and its criterion still say the file's `names` is kept as it is.
   - Evidence: D5 still says "a file that decodes keeps its `names` and drops its invalid `name`". The criterion still says "over a file holding an invalid name keeps the file's `names` and writes the new name". Under the new write step that `names` loses its invalid entries and gains the hostname rule's name.

Outside my lens: D5 now always counts the hostname rule's name as this machine's in the collision check. If another machine has recorded owners under this machine's hostname-rule name, the check passes and the sweep moves those owners. D5's shared-name bound does not list that case. The fix-claims or fix-proposal lens should weigh it.

I made no further tool calls.
