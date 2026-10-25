```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit. Rules checked: held or broken**

- **Allowlist is narrow and keyed (held).** `internal/lyxcwd/docslink_test.go:397-402` adds one entry keyed `(file, target)`. The key is `docs/research/codex-astra-solo-bench-2026-10-25/data/reviews/P4/opus-medium-r1.md` plus `pattern/PATTERN-owner-machine.md`.
  - It matches that one link in that one file. `allowlist.Allowed(docsLinkKey(relPath, target))` is exact-key (`docslink_test.go:~376`), with no glob or prefix.
  - The same target in any other file, or another target in this file, still counts as a break.
- **Self-expiring behaviour is intact (held).** `allowlist.Stale()` still feeds `t.Errorf("stale allowlist entry, delete it")` (`docslink_test.go:~414`). The entry is reported as deletable once the record is renamed or its link resolves.
- **Allowlist entry names its owner, per the comment at `docslink_test.go:393-396` (held).** `Why` names the codex-astra-solo-bench record and gives the reason: the link is a quoted verbatim reviewer line that resolves from the repository root.
  - The target file exists at `pattern/PATTERN-owner-machine.md`. The link is broken only because it is relative to the record's directory (`opus-medium-r1.md:20-21`).
- **Scope of the allowlist (held).** The grep shows `docsLinkAllowlist` is read at only one site, `docslink_test.go:411`, the `repo` subtest of `TestEnforcement_MarkdownLinks`. The D3 Location literal scan and the `writeTree` fixture subtests pass `nil` or their own lists, so the edit weakens none of them.
- **Parent merge touches no PATTERN entry (held).**
  - 9c167c500 adds 14 files, all under `docs/research/codex-astra-solo-bench-2026-10-25/`.
  - The `git diff --name-only` output I read shows `PATTERN.md` and the `lyxcwd`, `fabricengine`, `fabriccli` and `beamcli` files only as differences from the pre-merge base 9c167c500. They are the task's own settled change, not part of this amendment.
  - I did not re-read PATTERN.md for new declared-change gaps. The amendment adds nothing to it.
- **Commit message and departures (held).** 015a8b14e has no `Pattern-Departure:` trailer and needs none. It forbids no PATTERN entry's way, and the allowlist is a test-data edit.
- **Declared PATTERN changes landed (not re-judged).** That is settled-side scope. The amendment adds nothing that could undo it.

**Findings:** none.

**Tool calls (1):**
1. `grep -rn 'docsLinkAllowlist' internal/` plus `git diff --name-only 9c167c500..HEAD` — to confirm the allowlist has only one consumer, which my context lacked.
