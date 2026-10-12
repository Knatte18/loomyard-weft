# Fixer-report — webster round 1

## Changes

- F1: `internal/boardengine/store_test.go` — `keptBoardBody` task a now also carries `zz_quote` (a string with an escaped quote, `}`, `]` and `,`), `zz_nested_quote` (a nested object and array holding such strings and an escaped backslash before a brace) and `zz_esc` (a key spelled with an escape).
  `keptA` lists their compact values, so every `TestLoad` kept-key row asserts them after the save.
  No new top-level test.
- F2: `internal/boardengine/store.go` and `internal/boardengine/task.go` — removed the shared-lock claims from the `loadedStoreFile.UnmarshalJSON` and `eachMember` comments.
- F3: `internal/boardengine/migrate.go` — added `storedEntry.UnmarshalJSON`, which decodes into `decodedEntry` and sets `*e = storedEntryFrom(decoded, data)`; trimmed the `storedEntry` comment.
  `loadedStoreFile.UnmarshalJSON` keeps its single-pass path.

`lyx gate test ./internal/boardengine/... ./internal/boardcli/...` is green.
Each finding is its own commit.

## Disputed

None.

## Deferred

None.
