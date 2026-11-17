```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of the record's decisions that act on the same state, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D4 (refusing a tier whose `memory_mb` is below the per-test cap): held. Both state the same refusal, and the D4 criterion matches.
- D1 × D7 (detached runner and slot hold): held for keying, joining and slots. A low-severity finding on the wait cap is below.
- D1 × D2 (the package set includes packages that already have a map): held. A package whose last test was deleted is rewritten empty.
- D2 × D6 (one-ahead build, then checks after the build and before the write): held. A moved HEAD ends the call, and D2 cancels the pending build and waits for it.
- D2 × D7 ("replaced whole" against "a failed build keeps its old entries"): broken. See F1.
- D6 × D7 (failed build recorded on the map against "written only when clean and HEAD is still"): broken. See F2.
- D3 × D4 (tier env per test; tier 0 then tier 1, each at its own count): held.
- D3 × D5 (alone-rerun from the same coverage binary): held.
- D4 × D5 (a guard kill beside others reruns alone): held. The D4 and D5 criteria agree.
- D5 × D7 (exit judged after the alone-rerun): held.
- D5 × D6 (rerun before the write; second check before the write): held.
- D6 × D8 (last write wins between two runs): held.
- D6 × D9 (the dirty-tree refusal says commit or revert first): held.
- D4 × the `PATTERN-gate-slot` constraint (`-trimpath`; `-p` from tier 0): held.
- D2 × the `PATTERN-child-processes` constraint (cancelled build waited on): held.
- D4 × the `PATTERN-daemon-wakeups` constraint (guard sampling stays): held.
- The D7 green-skip criterion × D7: a wording gap. See F3.

**Findings**

F1 — LOW · consistency · D2 (`_lym/discussion/decision-record.md:59`) against D7 (`:136`)
- D2 says without exception that each package's map "is replaced whole".
- D7 says a failed build "keeps its old entries".
- D2 should name D7's failed-build case as its one exception, so the two do not contradict.
- Evidence: decision-record.md:59 and 136. The D2 criterion at line 201 holds only for a successful build.

F2 — MEDIUM · consistency · D6 (`decision-record.md:110-117`) against D7 (`:134-136`)
- D6's title and rule say a map is written only when its files are clean and HEAD is unchanged.
- D6 itself lets a build whose listing fails "record its failure with no check". D7 then writes that failure onto the package's map.
- Path: a dirty edit that breaks `go list`, or a HEAD that moved before the listing, gets a failed-build record written onto the map. That map carries the start commit and committed hashes.
- `lym gate redundant` then refuses the package as "recording only a failed build" (D9), although the tree at that commit builds.
- The HEAD check and the check of the package's own files need no linked set, and they could still run before that write.
- D6 does not say whether a compile failure after a successful listing gets the first check. The second check, before the write, is still stated.

F3 — LOW · consistency · D7 criterion (`decision-record.md:212`)
- The criterion reads "a gate call after it over the same tree runs its tiers rather than skipping them as green".
- A green record from before the map run still lawfully skips a tier (`PATTERN-gate-slot`, Gate carry), so the criterion holds only on a tree with no prior green record.
- It should state that precondition, or assert only that the map run adds no green record.

F4 — LOW · design · D1 (`decision-record.md:48-49`) against D7 (`:131`)
- D1 sends agent `--map` calls through the detached runner. D7 writes no run record.
- The detached front picks a call's long or short wait cap from the run records of its gate key (`internal/gatecli/detachfront.go:470-475`, `longWait` at 499).
- A map call has no gate key and no records, so it always gets the short cap. A `--all` map run that lasts minutes (D7 Bound) returns to the agent before it ends, and the agent has to call again to join it.
- The record says nothing about this interaction. A Decision should state the cap a map call waits under.

Tool calls:
- grep of `internal/gatecli/detachfront.go` for run-record and green reads: the manifest did not cover how the detached runner uses run records, which D7 drops.
