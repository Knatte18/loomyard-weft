```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each changed function or branch, I ran one concrete case, including an empty value and a failing call. I made no tool calls.

**Per function and branch**

- `timedNote` (`reviewstatus.go:40`): I gave it elapsed = 0 and elapsed = 61s. `Text` carries `0s` or `1m`, and `Steady` carries `<elapsed>` in the same spot, because `render` runs twice on one template. Correct. `Steady` is rendered from the placeholder, not found by replacing text, so a command or site string that happens to read like a time cannot corrupt it.
- `formatWaitNote` (`reviewstatus.go:45`): I ran it with an empty detail and a non-empty detail. With empty detail there is no parenthesis in either form. With a detail, the detail appears in both, so a real detail change still changes `Steady`. Correct.
- `gateWaiting`:
  - No records: it returns `next(st)` unchanged, error included. Correct.
  - Two records: the clauses are joined with `; ` in both forms, in the same oldest-first order.
  - Holders: the holders clause is added to both `Text` and `Steady`, so a holders change reprints. Holders list empty or failing: the clause is absent from both.
  - `next` fails: it returns the zero note and the error, as before.
  - `next` returns an empty `Text`: nothing is appended.
  - The closure reads `wait` inside the loop but runs immediately, so there is no capture hazard. Correct.
- `verifyWaiting`:
  - Paused marker: `timedNote` replaces the elapsed time and keeps the site. Correct.
  - Waiting marker: the holders text sits inside `detail`, outside the elapsed slot, so a holders change reprints and elapsed time does not. Correct.
  - Running marker: `formatWaitNote` takes `attempt N; <cmd>` as detail, so an attempt change reprints and elapsed time alone does not. This is the branch in the Seen log. Correct.
  - Marker not live: it falls through. Read error: zero note and the error. Correct.
- `shuttleWaiting`: I ran it with a live marker and with an empty detail. The kind stays in `Steady` and only the elapsed time is replaced. Not live: falls through. Error: zero note and the error. Correct.
- `parentLockWaiting`: the branch and holder slug stay in `Steady`, and only the elapsed time is replaced. Not found: falls through. Error: zero note. Correct.
- `reviewWaitingFor`: the chain order is unchanged and `noWait` returns the zero note. With nil `loc` it returns a nil hook, so the nil check at `checklist.go:167` still holds. Correct.
- `statusWatchLine`:
  - Status file unreadable or not found: line and key are the same unavailable string, computed per call from the label only, so it dedupes. Correct.
  - Hook error: the note is dropped from both line and key, which is the pre-diff behavior. Correct.
  - Empty `Text`: no suffix on either. Correct.
  - Wait starts or ends: the key gains or loses the ` | waiting …` suffix, so it reprints. Correct.
  - Row or state change: `RenderStatusLine` is the key's base, so it reprints. Correct.
- `PrintStatusLinesOnChange`:
  - The first poll prints regardless of key, because `printedAny` is false.
  - An unchanged key is suppressed, and the line printed is the first one of each run of equal keys.
  - Returning to an earlier key reprints, because it compares only against `lastKey`. Correct.
  - `polls <= 0` still loops forever, unchanged.
- `statusCmd`: `core["waiting"]` and `RenderStatusHuman` both get `waiting.Text`, so the envelope and human view are unchanged. A hook error still goes to the error envelope. Correct.
- `checklistPoll`: `waitNote = note.Text`, so the pane keeps the elapsed time. A hook error still drops the note. Correct.

**Findings:** none.

No tool calls were made; the context already held every definition and call site this lens needed.
