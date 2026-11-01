```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that each refusal, error, next step and test the fix added can be reached from the state it names and leads to progress.

**Ways forward the fix introduced**
1. D1: the runner verb refuses when its subdirectory has no lease, a passed lease or a process file, and names `lym gate test` or `lym gate card-verify` "with the same words".
2. D1: a runner that ends before it claims is a start failure, which the front reports with exit 1 and the runner's stderr.
3. D1 and D3: a runner killed without an exit file is superseded by the next front, and its next call starts afresh.
4. D3: a terminating signal to the runner is recorded as exit 128 plus the signal number.
5. D6: a `--wait` gate line reports its phase as running with no `slot_free` answer, and a lease cancel ends the line's process group.
6. New acceptance criteria under D1, D3, D5 and D6.

**Findings**

- **MEDIUM.** Changed text, D3: "A terminating signal to the runner cancels its call the same way, and its exit file records 128 plus the signal number."
  The replayed signal exit has no stderr line, so an agent cannot tell to re-run and reads it as a failed gate.
  The exit file is replayed to the next front that joins. For a SIGTERM from `runproc.End`, which the review cites at `internal/tendercli/deps.go:321`, that front replays exit 143.
  The lease abandonment gets "a line saying no caller came back", but the signal case gets no line.
  The implementer stencil (D9) treats only 75 with `done: false` as "run again", so 143 counts as a failure that no test produced.
  That contradicts D1's Bound and D5's "never a verdict".
  The way forward should be one added stderr line saying the runner was ended by a signal and the same command starts a fresh run, as D1 already does for a recorded 75.

- **LOW.** Changed text, D1: "The front that starts a run holds the key directory's lock until the runner's process file exists … a runner that ends before it claims is a start failure".
  Nothing bounds the time the front waits for the claim when the runner neither claims nor ends, for example when it hangs before its first write.
  The front then holds the key lock without end, and every other front of that key blocks on it instead of getting the 75 answer.
  The way forward needs a bound on the claim wait, after which the front reports the start failure.

- **LOW.** Changed text, D1: the refusal names the verb "with the same words".
  When the subdirectory or its lease is missing, the runner may have no record of the front's words.
  The words live with the front's request, and the review says a direct call has no front.
  So the refusal cannot always print the command it tells the caller to run.
  It should name the gate verb without promising the words, or the record should say where the runner reads them from.

No further tool calls made; my context held the changed decision record, the review and the rubric.
