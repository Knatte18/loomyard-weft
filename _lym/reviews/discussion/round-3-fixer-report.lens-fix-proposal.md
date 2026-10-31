```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals the fixer carried out as written (F1, F2, F3, F4, F6, F7).

1. **MEDIUM** — D4's Darn-Write missing-record bullet, "naming one way forward: `lym loom goto --to Record-Write` then `lym loom resume`" (F1).
   - **Issue:** This route is wrong for a run whose committed record was later removed from the working tree.
   - **Case:** A darn run passes Record-Write, and `decision add` commits an `Added after Discussion` entry. The working record is then deleted before Darn-Write starts. The pre-spawn missing-record refusal sends the orch to Record-Write, which writes a fresh record from the board entry. That record lacks the committed entry, so after Darn-Write's session the history check (D5) fails. `decision add`'s own historical-entry refusal (D4's "Missing a historical entry" bullet) also blocks putting the entry back.
   - **Evidence:** The fixer's own new D5 text says removing a file for Record-Write to write again "would drop" `Added after Discussion` entries, "and the history check would then refuse". The missing-record refusal is the same mechanism, and the F1 proposal ("one route for every run that reaches Darn-Write with no record") does not tell this case apart.
   - **Better route:** When HEAD in the `_lym` weft repository holds a committed record, restore it there and resume. Reserve the goto route for a run that never had one.

2. **MEDIUM** — D4 acceptance criterion: the added rows "one holding a fenced code block whose body has a ```` ```js ```` line … and one holding a fence indented three spaces" (F7).
   - **Issue:** These rows cannot detect whether the two new CommonMark clauses are implemented.
   - **Evidence:** They are board bodies inside Record-Write's outer fence. That fence is one backtick longer than the longest run in the body, so with a ```` ``` ```` run present the outer fence is 4 backticks. A 3-backtick ```` ```js ```` line or an indented ```` ``` ```` cannot close it under the length clause alone, so each row passes whether or not the info-string and indent clauses exist. The clauses only matter for top-level fences, which means a Discussion-written record.
   - **Better place:** Put the rows in D5's criterion, on a Discussion-written record with a top-level fence holding a ```` ```js ```` line, and with a fence indented three spaces.

3. **LOW** — D4's "Differs from its committed copy" bullet: "differs … only by appended `Added after Discussion` entries or `## PATTERN changes` bullets, the way forward is `lym fabric commit`" (F3).
   - **Issue:** The test cannot tell a failed `decision add` commit from a hand edit that appends an entry in the same form.
   - **Evidence:** The module doc of `internal/loomcli/decision.go` says the verb guards the caller rule: the claimed `--by` label, and `arbiter` refused except for the run's own arbiter. `lym fabric commit` would commit a hand-appended, mislabelled entry that never passed that rule.
   - **Consequence:** The fix should either bound the case to an appended record that the verb's validator accepts, or name the risk.

4. **LOW** — D4 and Scope still claim that the fence closes "by CommonMark's rule" (F7).
   - **Issue:** The stated rule still omits CommonMark clauses, so the claim is wider than the rule. CommonMark refuses a backtick opener whose info string holds a backtick, and it opens fences inside list items and blockquotes at their content indent. The new "a line indented four spaces or more neither opens nor closes a fence" contradicts the second clause for fences nested in a list item.
   - **Evidence:** The new D4 text lists only the indent, run-length and closing-trailer clauses.
   - **Effect:** This is the same class of gap F7 named, narrowed but not closed. It is harmless for Record-Write's own fence.

Tool calls:
- `grep` for AtomicWrite/CreateTemp/WriteFile in `internal/discussionparser`: I needed to know whether `decision add`'s append writes a `.tmp-` sibling that Record-Write's new cleanup could remove mid-write. It writes in place with `os.WriteFile`, and carry-over uses a `*.tmp` pattern, so there was no race finding.
