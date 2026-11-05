```yaml
lens: fix-claims
outcome: reported
```

Task: check, under the fix-claims lens, every bound, behaviour or code fact the fix added or rewrote against the code at HEAD.

**Findings**

1. **LOW.** D2 retention bullet, the rewritten text: "while it is delivered, expects a reply, has none and was posted less than the journal's seven-day retention ago".
   - **Summary:** the condition can never be true when retention runs, so the bullet describes a carry that never happens.
   - **Evidence:** `internal/creelengine/doc.go:68` says the writer "carries what is still live out of files older than seven days". A message in such a file was posted at least seven days ago, so "posted less than seven days ago" is false at every carry. A delivered message that waits for a reply is therefore never carried, and it lasts only as long as its own day file. That is about the lifetime the fix wants, but the sentence claims a carry the mechanism never does. The same flaw is in the reviewer's proposed fix (F7). A plainer statement: such a message is not carried, so it lives until its day file passes the seven-day retention.

**Claims checked and found to hold**

- D3: "`deliver` and `confirm` skip the sender resolution every other verb passes before dispatch (`internal/creelengine/server.go`)". `serve` calls `d.resolve` before `d.dispatch` for every request (server.go:208–213).
- D3: "an older binary's refusal of an unknown verb". `dispatch`'s `default:` returns `creel: unknown verb %q` (server.go:381–382). An older daemon may refuse a request even earlier, in `resolveSender`. Either way the answer is neither `accepted` nor `refused`, so the "any other answer is transient" rule covers it.
- D2 downgrade bound. Retention carries only the kinds it knows (unread or chaseable messages, queued injects, holds, program reads; doc.go:68) and then deletes the old files. So an older binary drops outbound events older than seven days without notice.
- D1 prune rule. `ReadShortname` returns `("", false)` on any read error (shortnamebinding.go:26–29), so the record can't tell an unreadable file from a missing one. The new rule drops an entry only on a missing path or a found, different shortname, so it never relies on that difference.
- D3 bound "tells a caller nothing it could not learn by reading the sending hub's journal". The journal files belong to the same user, and the answer is only yes or no to a full tuple that includes the body hash.

**Tool calls**

- `sed`/`grep` on `internal/creelengine/server.go` (serve/resolve) and the retention lines in `doc.go`: my context had only the grep line numbers for `resolve`, not the dispatch order, and none of the retention text.
- `grep` for `dispatch`'s default case and `externalFromClaim`: my context did not show how an unknown verb is answered.
