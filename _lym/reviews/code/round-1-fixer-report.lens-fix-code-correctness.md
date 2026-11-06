```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check F1's change set (`receive.go` `noticeDropLocked`, the new `crosshub_test.go` row) through fix-code-correctness.

**Traces**
- **First drop, post fails:** `window.notices` is 0 and `undisclosed` is 1, so `noticesBefore` = 0 and `counted` = 1. The failed post restores `notices` to 0 and `undisclosed` to 1.
- **Next drop, post succeeds:** `undisclosed` becomes 2 and `counted` is 2, so the body carries "2 drops … since the last notice". The window's notice count is correct.
- **Fifth slot fails:** `notices` goes back to 4. The next drop becomes the fifth and appends the "further drops … without a notice" text again, which is correct.
- **Callers:** `noticeDropLocked` has one caller, `dropLocked`. Its signature and return are unchanged.
- **Test against the old code:** the second notice would carry `counted` = 1 and omit "2 drops", so the row fails with the defect in place.
- **Test against a stray failure:** the row asserts a `refused` answer with a nil error after the fault. That pins the failure to the notice write: the high-water record and the drop's own write both passed.

**Findings**
- **LOW**, `receive.go` doc comment of `noticeDropLocked`, "leaves the window as it was":
  - The window is not left as it was. The failed drop has already been added to `undisclosed`, and a new or rolled-over window may have been created.
  - The code is right, but the sentence overstates it. "Does not spend a notice slot, and keeps its drops counted for the next notice" would be accurate.

**Tool calls:** none. My context already held every changed file.
