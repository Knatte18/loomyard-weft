no findings

- F2 trace: issue 1 with `Comments > 0` and a failing comment fetch now emits `issue #1: github issue comment list failed on <repo>: boom` through `outputError`; the success and compact paths are untouched.
- F3 test: the `CommentListFailure` row requires `issue #1: github issue comment list failed`, so dropping the prefix or failing in an earlier step fails it; rows with an empty `wantErr` still check for a non-empty message.

Tool calls (1 of 6):
1. `grep -n -A25 "func (r *Repo) classifyCallError" internal/hubissues/hubissues.go`: read the exact error text, which the context lacked.
