```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked each claim the fix added to cards 8, 7 and 1 against the code at HEAD.

No findings.

- **`os.ReadFile` of a directory (cards 7 and 1).** On Linux, opening a directory read-only succeeds, and the read then fails with `EISDIR` for every user, root included. `errors.Is(err, fs.ErrNotExist)` is false for that error. So both rows fail without a skip and tell an unreadable file from an absent one, as claimed. Card 1's statement agrees with its own rule: a read error of `path` returns the error.
- **`sleep 120; true` under a 100ms `Timeout` returns before the 60-second deadline (card 8).** With the group kill, the forked `sleep` dies along with the shell. Without it, `hookWaitDelay` (two seconds) still ends the wait. Either way the call returns long before 60 seconds, so the claim holds.
- **The marker row pins the group kill (card 8).** With the group kill, the subshell dies at about 100ms and never touches the marker. Without it, the subshell survives the shell's kill and holds the stdout pipe. The call then returns at about 2.1s through `WaitDelay`, and the subshell touches the marker at about 2s. The test checks at about 5.1s and catches it. So the row fails without the group kill and passes with it, as claimed.
- **`hookWaitDelay` (card 8).** The name is new and nothing in `internal/shuttleengine` defines it or sets `WaitDelay`, so it clashes with nothing.

Tool calls:
- I searched `internal/shuttleengine` for `hookWaitDelay` and `WaitDelay` to rule out a name clash, since my context held the plan but not the package source.
