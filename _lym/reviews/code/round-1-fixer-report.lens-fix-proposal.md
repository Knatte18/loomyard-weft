```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (reword `doc.go` line 114 to "a session that is no orch of the entry's beam") against `runsettings.go`.

**LOW** — `internal/boardcli/doc.go:114-115`, "The refusal of a session that is no orch of the entry's beam names the operator, the hub orch or the orch of the entry's beam as the way forward."
- **Summary:** The qualifier is still too broad, because it also covers a case that gets the new parent-orch refusal.
- **Evidence:** In `runsettings.go:116-118`, an orch whose slug differs from the entry's beam gets `refuseBeamOrchOutsideBeam`, which names the parent orch. That session is also "no orch of the entry's beam".
- **Concrete case:** The orch of beam `A` writes an entry under beam `B`. It is no orch of the entry's beam, so the sentence says the refusal names the operator, the hub orch or `B`'s orch. The code (`:127-136`) tells it to post to `A`'s parent orch.
- **Why it still holds:** Lines 115-120 fix the contradiction for a reader who reads on, but line 114 alone is still a partly wrong rule, the same defect F1 raised.
- **Better wording:** "The refusal of a session that is not an orch names ...". That matches the code's actual split at `runsettings.go:100`, where only a non-orch role gets the old text.
- **Why the code's own wording doesn't excuse it:** The error string at `runsettings.go:99` uses the same "no orch of the entry's beam" phrase, but only a non-orch session can reach it.

Calls made:
1. Read `internal/boardcli/runsettings.go:80-150`, because my context held only `doc.go` and not the refusal code needed to test the proposed wording.
