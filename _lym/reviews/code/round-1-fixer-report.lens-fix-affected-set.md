```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the F3 and F4 site searches and found one LOW.

**LOW** — `internal/runprofile/runprofile.go:38`
- The comment says each profile "is one whole entry set from a JSON object". A dotted key can now set a value below a profile, so the sentence is no longer the whole story.
- It describes what a profile is, so it is not strictly false.
- This was outside the review's F3 location but inside F3's set (the `JSONEntries` wording), so an update is optional.

Everything else in the sets is already updated and agrees with the fix:
- **F3 (the `JSONEntries` wording):** `internal/configengine/doc.go:27`, `internal/configengine/set.go:48` and `:110`, and `internal/yamlengine/doc.go:13` all carry the dotted form. `internal/configcli/doc.go:129-137` covers it in its dotted paragraph.
- **F4 (the dot rule):** the sentence is in `internal/configcli/doc.go` and the `SetEntries` doc comment (`internal/yamlengine/set.go`). `SetEntries` is called only from `internal/configengine/set.go:121`. That call has no other doc site that needed the sentence.

Tool calls made:
- grep for the `JSONEntries` / whole-entry wording across Go and md files: to rerun the F3 set search, since my context held only the files already read.
- grep for `SetEntries` / `a JSON object`: to rerun the F4 set search for other doc sites and callers, for the same reason.
