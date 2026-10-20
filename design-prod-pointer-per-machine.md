# prod pointer is shared across machines while each binary is local

`prod-pointer-per-machine` · Note · enhancement, deploy

## Seen (2026-10-20)

- `update-plugins.sh` (`tools/deploy/main.go`) builds `lyx` into the machine's Go bin dir, syncs the plugins, then pushes `HEAD` to `origin/prod`.
- `prod` is one pointer on origin, shared by every machine, while each machine's installed binary is local.
- On the Kubuntu machine a deploy moved `prod` to `a1f435dc`; the work laptop still ran a binary built from `c03af73d` while `prod` claimed `a1f435dc`.
  Nothing on either machine shows that the laptop is behind `prod`.

## Idea

`prod` cannot say what a given machine runs, so it is the wrong record for "what is deployed".
Options to weigh:

- Keep the deployed commit per machine (a local stamp beside the binary, which `lyx` already knows as its build revision), and let `prod` mean only "newest commit cleared for production".
- Have `lyx` (or `tender status`, or orch start) warn when the installed binary is behind `origin/prod`, naming `update-plugins.sh` as the way to catch up.
- Drop the pushed `prod` pointer altogether if nothing needs a shared one.
