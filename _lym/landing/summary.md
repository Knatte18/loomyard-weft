# Class a remote whose local repository path is missing as unreachable

An origin that is missing on disk took a different route from one that refuses a connection.
A push to a removed bare repository was recorded with failure class `other`, which stops the run,
and an origin reconcile against a remote naming a missing directory returned an error instead of reporting origin unreachable.
A missing local origin, such as a removed directory or an unmounted share, can come back as a refused connection can,
so both now treat it as unreachable.

## The change

`gitexec.IsTransportFailure` keeps its network phrases and gains one more case.
When a remote subcommand's stderr says the repository it named cannot be opened,
git's `'<path>' does not appear to be a git repository` for push, fetch, pull and ls-remote,
or `repository '<path>' does not exist` for clone,
it reads the named path, resolving a relative one against the directory git ran in, and strips a `file://` prefix.
When that path does not exist on this machine, the failure is a transport failure.

Every caller that classifies through this function follows:

- the push classification records an owed `transport` failure, retried quietly, in place of `other`;
- `ReconcileOrigin`'s ls-remote retry loop retries the failure and then sets `Unreachable` with a nil error.

## Design decisions

- A path that exists but holds no repository stays a refusal.
  That is a misconfigured remote, which retrying does not fix, so it still stops with `other` or an error.
- The check is a stat of the path git names, not a lookup of the remote's URL, so it adds no git call to the failure path.
  Bound: an ssh remote naming a server path that is also absent on this machine reads as unreachable.
  Its push is then retried as owed instead of stopping.
  GitHub over ssh prints `Repository not found` instead and is unaffected.

## Tests

- `TestIsTransportFailure` gains hand-built rows: a missing absolute path, a missing `file://` URL,
  a missing relative path read against the command's directory and a clone of a missing path count;
  an existing directory holding no repository and a local command do not.
- `TestIsTransportFailure_RealGit` removes the bare origin and pins git's real stderr for push, fetch, ls-remote and clone,
  then recreates it as an empty directory and pins that a push there is not a transport failure.
- In fabricengine, a push of a never-pushed branch to a removed local origin writes one `transport` record and one mail.
  The `other`-class push test now points origin at an existing directory holding no repository.
- The reconcile lock test now asserts `Unreachable` per row.
  Its missing-directory row expects no error and `Unreachable` set,
  and a new row for an existing directory holding no repository expects the error.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
