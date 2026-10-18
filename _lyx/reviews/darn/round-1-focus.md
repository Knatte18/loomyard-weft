---
round: 1
focus:
  - "internal/creelengine/deliver.go and reminder.go: does the snapshot split, worker.typeNotice and Store.markNoticeRead carry mail only for the hold owner, once per probe, after any due inject and before the reminder, and never journal a message read twice when a concurrent read holds it reserved?"
  - "Carried-mail eligibility and rendering: does isNotice/composeNotice match the board entry and its Added after Discussion entries (sender classes, 4096-byte cap, control-character collapsing, id-first line), and does the reminder's count, ids and backoff cover only the remainder?"
  - "Store.Read and lyx creel read in internal/creelcli: are the dropped --all flag and request field removed everywhere (callers, stencils, docs, tests), and do the header rules, the five-minute posted rule, --envelope and refusal JSON behave as the summary claims?"
  - "Scope against the board entry: does the diff change anything the amended entry does not license, such as the stencil edits, the sandbox suite changes and the tmux/llm smoke tests, and does it omit anything the entry asks for?"
  - "Doc and stencil landing: do creelengine, creelcli, orchcli and orchengine doc.go files plus PATTERN.md or docs/overview.md changes match the new behavior, and do the orch and handoff stencils drop or weaken any rule the entry did not ask to change?"
  - "Test economy: is each new behavior covered once, with no duplicate of an existing test and no assertion on stencil prose, and does the summary's claim about the changed returned-undelivered rows hold against the diff?"
---

Seed focus from the change description and the board entry; the reviewer checks each claim against the diff.
