```yaml
lens: code-correctness
outcome: reported
```

Task restated: lens code-correctness over the `machinecli`, `boardengine` and `machinename` changes. I ran one concrete case per function or branch.

**Per function or branch (case run, verdict)**

- `writeOwners` replace selection:
  - Replace `{"box","","old"}` after a pull that brought in `oldbeam` (box), `farbeam` (far) and `shipped` (box, done): selects the prime and `oldbeam` only. Correct.
  - An empty entry never matches, because `replaced` requires `owner != ""`. Correct.
  - A done or non-beam entry is skipped. Correct.
  - The replace loop runs after the Slugs loop and appends the prime only once. Correct.
- `SetOwners` guards and commit subject:
  - Replace together with Slugs, Prime, Claim or Takeover is refused. Correct.
  - Replace `{""}` alone passes the guard, writes nothing, and the subject falls back to "for ". Correct, if cosmetic.
  - Replace `{"box","old"}` gives "owner neve replacing box, old". Correct.
- `SetOwners` post-write read:
  - A replaced slug whose standing owner differs (competing origin write) goes to `Lost` with that owner. Correct.
  - A write with nothing selected leaves `written` empty, so no commit, and `PushRecorded` still pushes an earlier unpushed commit. Correct, and it matches the "pushed" outcome.
- `nextFile`:
  - No file, hostname `box`, new `osl46`: `[box, osl46]`. Correct.
  - Hand-written `{"name":"osl46"}`: `[box, osl46]`. Correct.
  - Invalid name with names `[neve, osl46]`: `[neve, box, osl46]`. Correct.
  - Undecodable file: `[box, osl46]`, names lost. Correct.
  - Empty entries are dropped and counted. Correct.
  - Hostname error: no hostname added. Correct.
  - New name equal to the hostname name with no file: `[name]`. Correct.
- `needsWrite`: already-set file with identical `names` writes nothing. Correct. A missing, undecodable or invalid-name file always writes. Correct.
- `ownNames`: an undecodable file gives the hostname name only. A bad-name file keeps the decoded names. Correct.
- `checkHub`:
  - Pull error or skip gives `CheckedLocally`. Correct.
  - Unpushed read error is noted and `ahead` becomes false. Correct.
  - `ahead` is read before the pull, as D6 asks. Correct.
  - Unreadable board refuses, naming the hub and the re-run. Correct.
  - A collision is detected on the prime or an entry owner that is not among the machine's own names, including done entries and origin-only owners (the pull comes first). Correct.
  - Own name skips the collision check. Correct.
- `sweepHub`: a write error gives `outcomeFailed` with the re-run as the way forward. Correct. See F1 for the `Lost`-only case.
- `setName` ordering: the name check, then the file, hubs and checks, then the file write, then the sweep, matches D5. Replace is `file.Names` minus the new name. Verified: a rename back to a former name gives `[osl46]`, and a re-run after the file was written still replaces the old names. A write failure after the checks returns an error and sweeps nothing. Correct.
- `runSet`: a failed hub or a `Lost` entry gives a non-zero exit with the same fields. Correct.
- `readCurrent` and `showName`: all the bad-file branches print the error and the file path. A hostname error with no file prints the hostname error. `former` is `[]` for a hostname source. Correct. `showName` writes nothing.
- `hubsToSweep`: the registry's hubs are deduplicated and the cwd hub is added last when absent. A cwd hub in the registry is not repeated. Correct, but see F2.
- `FromThisHost`: delegates to `nameFromHostname`. Correct.
- `noDirectoryRefusal`: on Linux, `XDG_CONFIG_HOME` is skipped when relative and a relative `HOME` errors in `locate`. Message says "XDG_CONFIG_HOME or HOME", which is correct, though a non-set `HOME` with a relative `XDG_CONFIG_HOME` still reads as unresolved. Correct.
- `state.WriteJSON` creates the parent directories, so a fresh config directory works. Correct.

**Findings**

- F1 — LOW — design — `internal/machinecli/set.go:193-209`, evidence `internal/boardengine/ownerset.go:156-180`.
  - When every selected entry comes back in `OwnerWrite.Lost` (a competing origin write took them all), `Recorded` is empty, so `sweepHub` leaves the outcome as `nothing to sweep`. `setName` then upgrades it to `pushed` if the board was ahead.
  - The `lost` list and the non-zero exit still appear.
  - The outcome label contradicts the lost entries, and `pushed` hides that the sweep did not finish.
  - Fix: in `sweepHub`, set a distinct outcome (for example `outcomeKept`, or leave `swept` with an empty `Slugs`) when `len(written.Lost) > 0`, and skip the `pushed` upgrade for it.

- F2 — LOW — design — `internal/machinecli/hubs.go:44-62`.
  - `hubsToSweep` deduplicates by string equality only. The registry path and `lymcwd.ResolveWorktree().HubPath` can differ by symlink or case form for the same hub.
  - That hub would be checked and swept twice. The second sweep reports "nothing to sweep" and is harmless, but the output lists the hub twice.
  - Fix: compare through `filepath.EvalSymlinks` (falling back to the raw path), or accept this as a bound. This is a judgment call.

No BLOCKING or MEDIUM defects found in the lens.

**Tool calls**
1. Read `internal/state/state.go` and `internal/standalonestate/standalonestate.go` (grep for `WriteJSON` and `AbsoluteUserConfigDir`): my context lacked the directory-creation behaviour of `WriteJSON` and the exact variable resolution behind the no-directory refusal.
