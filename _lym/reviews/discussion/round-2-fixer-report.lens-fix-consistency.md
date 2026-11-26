```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 change set agrees with its sibling decisions, criteria and cross-references (fix-consistency lens).

**Findings**

1. **LOW.** D2's `lym area init` sentence no longer covers everything the area file holds.
   - **Changed text:** D2's new file contents ("an optional `tmux` path, and the hub's `shortname` and `repository` (D8)") and D8's new bullet ("`lym area init` writes the area file's `shortname` … the operator writes its `repository`").
   - **Summary:** D2 still says `lym area init` "writes it in full the first time, with the read, exec and write lists D6 defines". It names neither `shortname`, which D8 now has init write, nor `repository`, which D8 leaves to the operator. No decision says what `lym area start` or a receiving daemon does when `repository` is absent.
   - **Evidence:** D2 "`lym area init` writes it in full the first time, with the read, exec and write lists D6 defines". D8 "`lym area start` copies both into the entry". D3's start check refuses only "a file failing D2", and D2 sets no rule for these keys.

2. **LOW.** D6 still calls the area state directory "read-only inside", but no area now has any read grant on it.
   - **Changed text:** D6's read list, which dropped "every hub's supervisor record (D3)", and D3, which dropped "D6 grants read on every hub's supervisor record alone."
   - **Summary:** The supervisor record was the only read grant inside the area state directory. D6's socket bullet still says the directory is "read-only inside", which now overstates what the area can do there. "Unwritable inside" would match the ruleset.
   - **Evidence:** D6 "The supervisor's socket lives in the area state directory, read-only inside, so no area process can unlink or replace it". D6's read list is now "`/proc`, `/sys`, `/dev`, `/run`, the machine's hub record and the token file (D11)".

I checked these and found them consistent:
- D13's and D15's verb lists now name D6's `attach-scope`.
- D9's scope names fall under D3's prefix.
- D3's boot and Bound, D13's `window` and its criteria all use "canonical parent is the hub root".
- D12's Bound matches D16's relay non-claim.
- D7's corrected sentence matches D8's new bullet.
- D15's criterion now covers a new area hub starting.

Further tool calls: none.
