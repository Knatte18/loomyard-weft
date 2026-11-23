---
round: 1
focus:
  - "internal/output/output.go ErrorEnvelopeLine: does it refuse a legitimate body that quotes an error envelope line, such as a task about CLI errors, and do its callers (Record-Write, applyBodyFile) leave a way to store such a body?"
  - "internal/loomshed/recordwrite.go Call: the new Stuck branch against the missing-entry branch above it; is cancelErr called the same way, is nothing written, and does the reason's way forward (batten cancel, board upsert --body-file, restart) match how this run's board lock behaves?"
  - "internal/boardcli/cli.go isBareSlug and resolveLookup: does the new bare-slug reading shadow any input that parsed before, such as JSON literals (null, true) or handles, and does it match the slug shape the board accepts elsewhere?"
  - "internal/boardcli/bodyfile.go applyBodyFile: do upsert and merge both reach the new refusal, and does any other body path, such as the inline payload body or stdin, still store an envelope unrefused?"
  - "Scope against the decision record Goal: it asks Record-Write to refuse a non-body Goal and the upstream step that fed the error to be traced; does the bare-slug change in cecf8b2b9 trace that step, and is anything beyond it unlicensed?"
  - "Commit trailers: do the Root-Cause trailers of 3cbd45a96 and cecf8b2b9 name defects the diff removes, and does the loomshed seam_enforcement_test.go change widen a check at the base without a Loosened-Check trailer?"
  - "Tests: TestApplyBodyFile, TestRecordWrite_Call, TestErrorEnvelopeLine and the TestCLI bare-slug case; do they assert independent literals, and is the changed bodyfile_test.go case still checking what its name states?"
---

Seeded from the diff and the board entry in the decision record's Goal; no prior round exists.
