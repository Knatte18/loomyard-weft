# Trial creel mail across hubs

`creel-cross-hub-trial` · Note · research

A darn `docs` run in which the loomyard orch drives a run in `lyt` from start to landing through creel alone, with the operator doing nothing, recording the result under `docs/research/`.

Setup before the first cross-hub post:

- `trusted_hubs: [ly]` in the `creel` config of `lyt` and of `cs`, set by the operator in those hubs.
- A deployed binary carrying creel-cross-hub-build.
- A restart, right after the deploy, of every session of the machine's hubs launched before it. Until a session restarts, an orch strand of another hub reaches it under the old trust sentence.