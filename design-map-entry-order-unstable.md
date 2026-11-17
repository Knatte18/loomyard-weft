# Coverage map rewrites funcs in another order on each build

`map-entry-order-unstable` · Note · bug

Found by the map-tooling-trial.

Two `lym gate test --wait --map internal/boardengine` runs on one commit (`b92c95ee`) wrote maps that hold the same sets per test but not the same bytes.
Beside `built_at` and `measured_at`, the `funcs` array of four tests lists the same ids in another order: `TestHandles_AssignedLowestFreeAndStable`, `TestHoldsRaddleFiling`, `TestRunLock`, `TestSetStatusReleasingProject_ReleasesDraftMembers`.
Set difference per test is empty in both directions, so only the order is unstable.
Two `internal/durfmt` runs differ in the timestamps alone.

A map that rewrites with a spurious diff each build makes any map committed or compared by bytes churn.
Sort each test's `funcs` and `blocks` before writing.
