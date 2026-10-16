# creel log shows nothing for mail carried forward from an old day file

`creel-log-carried-mail` · Note · bug, creel

Imported from [issue #684](https://github.com/Knatte18/loomyard/issues/684).

Seen: the journal holds `{"event":"carried","id":900,...}` and the receiver `inbox` counts message 900, but `lyx creel log --id 900` returns `[]`.

Expected: `log` shows a carried message, at least as a `carried` event.

Found by the creel stage 1 re-test of 2026-10-16 (sandbox scenario C10).