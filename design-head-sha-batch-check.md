# Resolve head_sha prefix candidates in one git call

`head-sha-batch-check` · Note · enhancement, webster

## Seen

The PR-Gate review of `webster-record-batch` (PR #473) found that `commitsNamedBy` in `internal/websterengine/gitwrap.go` runs one `git cat-file -t` per object a short `head_sha` prefix names, and a 4-character prefix in a large repository can name many.

## Idea

Classify every candidate in one `git cat-file --batch-check` call.
