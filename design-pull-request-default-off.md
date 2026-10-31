# Runs land without a pull request unless the hub turns it on

`pull-request-default-off` · Note · enhancement, dobby, landing

## Seen (operator, 2026-10-31)

The beam module's `pull_request` key defaults to `on` in its template (`internal/dobbyengine/template.yaml`), so a hub that has not set it lands every run through a GitHub pull request and the PR-Review row.
The operator wants no pull request unless a hub asks for one: the default is `off`.
On 2026-10-31 the loomyard hub's own `pull_request: off` was lost (issue #907), and the template's `on` took over at once.

## Change

- The template default of `pull_request` becomes `off`, with its comment, the package docs and the tests that pin `on` brought in line.
- `on` and `required` keep their meaning for a hub that sets them.
