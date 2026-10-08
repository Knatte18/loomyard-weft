# Git push without a timeout hangs a batten run after laptop sleep

`git-network-timeout` · Note · bug

Imported from [issue #458](https://github.com/Knatte18/loomyard/issues/458).

## What happened

A `lyx batten run review-lens-fan` began a best-effort status push to the weft remote at 14:11:21; the laptop hibernated four seconds later.
After resume the push hung on a dead HTTPS connection (`git send-pack` in `futex_do_wait`) for 2.5 hours, and the batten run sat at Run-Shed without starting the child loom, with no notice to the parent.
Killing the `git push` let batten log `status push failed, next transition will catch up` and go on.

## Expected

Git network calls from lyx (push and fetch, best-effort or not) run under a timeout, or with `http.lowSpeedLimit`/`http.lowSpeedTime` set, so a dead connection fails within minutes and the best-effort path takes over.

