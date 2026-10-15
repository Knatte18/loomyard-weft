# creel: log, inject help, sender field and release output inconsistencies

`creel-output-inconsistencies` · Note · bug, creel

Imported from [issue #664](https://github.com/Knatte18/loomyard/issues/664).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.
Three small output and help inconsistencies in `lyx creel`, filed together.

1. `lyx creel log` prints each `reminded` event with `"id":0` and without the message ids, although the journal record carries them (`"unread":[22]`) and the backoff `step`.
   A reader of the log cannot tell which messages a reminder named.
2. `lyx creel inject --help` says "Only the orch, a program and an external caller may inject; any other strand posts instead."
   The design (`internal/creelengine/doc.go`) and the code let any strand inject into its own pane, and `lyx creel inject ctest:dave "self inject"` run from dave's pane is accepted.
3. The waiting verbs print an empty sender: `lyx creel inject --wait` and `lyx creel post --wait-reply` print `"sender":""`, where the plain verbs print `"sender":"ctest:EXTERNAL"`.

Also seen: `lyx creel release <unknown name>` answers `{"ok":true,"released":false}` instead of naming the strands that exist, as `hold` does.

## Expected

The log shows the reminded ids (and step), the inject help matches the privilege rule, the sender is printed the same way by every verb, and `release` refuses an unknown receiver like the other verbs.
