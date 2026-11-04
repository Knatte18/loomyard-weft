```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran the changed branches and added test rows against the defects they pin.

**Findings**

1. **LOW**: `gitWritingOption` in `bashlookup.go`.
   - **Summary:** `gitReads` misses git's unambiguous option abbreviations and bundled short flags.
   - **Evidence:** git's option parser accepts a unique prefix, so `git diff --out=x` and `git grep --open-files less foo` write a file or run a program, yet `strings.HasPrefix(arg, "--output")` and `"--open-files-in-pager"` do not match them. `git grep -nOless foo` bundles `-O` behind another flag and is not caught by `HasPrefix(arg, "-O")`. `--ext-diff` is an exact match only, so `--ext-d` passes.
   - **Impact:** the effect is still one miscounted lookup, as the review said of F2, so this is a leftover of the F2 premise that the fix did not close. I did not run git against these forms; this rests on git's documented parser behaviour.

2. **LOW**: the new row `SubagentMainChainLineIsNotCountedInTheChain` in `batchingnudge_test.go`.
   - **Summary:** the row pins the agent side of `tailMessages` only, not `latestReading`, which F8 also names.
   - **Evidence:** `writeNudgeTranscript` gives every line of a transcript the same `tokens` value. A main-chain line in an agent transcript therefore yields the same reading whether `latestReading` skips it or not, and the row cannot fail on a `latestReading` defect.
   - **Impact:** the `tailMessages` half of F8 is pinned. For the `latestReading` half, a row would need a main-chain line with a different token count. I did not see `latestReading`'s source in my context.

**Checked, no defect**

- **F1 row (`BashCallThatIsNoLookupLeavesTheStateUnread`):** without the early return, `UpdateJSON` reads the corrupt file and returns an error, so the row fails at `t.Fatalf("BatchingNudge() error")`. The new `if tt.corruptState { return }` sits after the nudge assertions and before the state read, so the existing `CorruptStateFileIsAnError` row is unaffected because it returns earlier through `wantErr`.
- **F2:** `gitReads` returns false for `git diff --output=x`, `git log --output x`, `git grep -O less foo` and `git log --ext-diff`. Plain `git log --oneline -5` and `git -C /repo show HEAD` still return true.
- **F3:** dropping `toolBash` from `batchingNudgeMatcher` fails the new assertion.
- **F5:**
  - Each new true and false row is correct by trace through `splitShellSegments`, including `\;` in the `find` rows, `|&amp;`, `&gt;&amp;2` and `&amp;&gt; out.txt`.
  - Each false row fails when its table entry is removed. `git -C` with no directory panics on `args[2:]` if the length guard is removed, so that row fails too.
  - `sed --quiet --in-place=.bak 5p x` includes `--quiet`, so the row fails without the `--in-place` branch.
- **F6:** for the backslash, `.` and `..` ids, the fixture writes the agent transcript at the path `chainTranscript` would build without the guard, so each row fires a nudge unless the guard stays. `.` and `..` rely on `subagentTranscriptPrefix` being non-empty, so their file names stay valid.
- **F7:** the row seeds the agent's `NudgedMessage` as `msg-2`, the current message, so it is silent. Reading the main count's empty `NudgedMessage` instead would nudge and fail the row.
- **F8:** the row is silent and passes only if the main-chain line is skipped. The run would be 3 against `LookupRun` 3 if the line were counted.
- **F4 and F9:** the text edits change no code. No other copy of the help text exists in the repo, so no golden file or doc is left stale.
- **Callers:** `gitReads`'s signature is unchanged, and `lookupChain` and `kindOf` are unaffected.

**Tool calls**
1. `grep "recent messages against the"` across the repo: to see whether a golden file or doc held a copy of the `hookbatching.go` help text that F9 changed; my context held only that file.
