# One-repository hubs redesigned: records on their own branches, the layout known only to fabric

`single-repo-records-branches` · Task · feature, fabric

{"error":"invalid json: invalid character 's' looking for beginning of value","ok":false}

## Scope (operator, 2026-11-19)

The one-repository hub is for the operator's own use only.
The two-repository hub stays the standard and the one used with other people: every trace of LoomYard lives in its own repository there.
Extra branches in a one-repository hub are clutter to anyone else reading the repository; Discussion weighs keeping records and the board under a custom ref namespace such as `refs/lym/...`, which no default clone fetches and GitHub lists as no branch, against plain branches.
