---
round: 1
entries:
  - key: batch-upsert-accepts-arras
    status: closed
    finding: F1
    reason: "875e74ff1: UpsertTasksBatch in internal/boardengine/store.go runs checkOfferedRecipe against the pre-batch s.tasks entry, and TestUpsertRefusesTheArrasRecipe drives the batch write for a new and a darn entry"
---
