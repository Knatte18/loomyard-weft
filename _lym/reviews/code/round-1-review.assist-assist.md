---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: contracts/orchrole/role-intake.md:36
    summary: Comment-note upserts bypass the issue deduplication that makes overlapping intake forks harmless
    fix: In the comment-splitting rule, require the fork to import the issue before creating split notes and create them only when its import reports a board write, including an error reporting a completed board write followed by a close failure. Explicitly skip splitting when import returns recorded. Preserve the existing close retry rule.
    origin: lens:code-affected-set
---

### [MEDIUM] Overlapping forks can duplicate comment notes

**Location:** contracts/orchrole/role-intake.md:36

**Issue:** Two forks can read the same issue and independently choose different slugs for its separate comment finding.
The new rule directs both to upsert a note, even when the second import returns `recorded`.
The existing concurrency guarantee at lines 17–20 relies on import settling the issue once.
`Store.UpsertTask` in `internal/boardengine/store.go:590` identifies entries by slug;
different slugs therefore create separate notes without issue-level deduplication.

The list fails without partial output when a comment fetch fails, and compact mode fetches no comments.
The shared filter preserves import behavior and comment order, and full output initializes empty comments as an array.
Import already carried comments under headings at the review base.
The output documentation and per-issue report extension otherwise agree with the requested behavior.
The changed request assertion preserves compact mode's prohibition while allowing the new full-mode requests.

Validation: the focused intake test invocation could not acquire a gate slot because the shared gate directory was read-only.
No test result is claimed from that invocation.
The supplied reach and redundancy reports were considered as review evidence.
