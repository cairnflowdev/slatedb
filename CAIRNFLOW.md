# SlateDB fork for Cairnflow

This branch is a fork of [SlateDB](https://github.com/slatedb/slatedb) `v0.18.0`
with patches that [Cairnflow](https://github.com/cairnflowdev/cairnflow) needs.
It is licensed under the Apache License 2.0, like
SlateDB; see `LICENSE`. The files listed below were modified by the Cairnflow
authors; every other file is unchanged from `v0.18.0`.

We intend to upstream these patches and drop the fork. Status per patch is
below; "fork only" means the change exists only to serve Cairnflow and has not
been proposed upstream yet.

## Patches (in order, on top of `v0.18.0`)

| Patch | What it does | Upstream |
|---|---|---|
| Probe the real WAL tail before a clone accepts a source | A clone of an open database checked only the WAL IDs in the stored manifest and could drop writes already durable in the WAL. | fork only |
| Project a clone source on more than one key range | `CloneSourceSpec::projection_ranges`: a source can keep several key ranges (a tenant spread over several region-tagged ranges). | fork only |
| Wake the in-process compactor and writer with signals instead of polls | Compactor coordinator, worker and writer in one process notify each other instead of waiting for poll ticks. | fork only |
| Copy the WAL tail when a clone takes the latest state | A clone without an explicit checkpoint now copies durable WAL entries written after the manifest's WAL range. | fork only |
| Run the WAL flush timer only while the current WAL holds entries | Removes idle wake-ups (many idle databases in one process). | fork only |
| Add WAL names, a shared-WAL GC contract, and a streaming DbReader | Additive extensions of the pluggable WAL (RFC 0030): a WAL name in the manifest (`ManifestV2` field 18), GC hooks for a WAL shared by many databases, a streaming reader. | issue planned for the manifest field |
| Reclaim jobs of dead workers on the commit tick | A dead worker's compaction job no longer blocks L0 compactions until the next manifest poll. | fork only |
| Let an in-process garbage collector skip passes while idle | `GarbageCollectorOptions::idle_skip_max`. | fork only |
| Export `estimate_visible_size` of `SsTableView` and `SortedRun` | Size estimates for projected clones from outside the crate. | PR drafted |
| Name the final checkpoint of a clone after the clone path | Lets a cleanup tool find the checkpoint a failed clone left on its source. Plus its test. | PR drafted (test to rewrite for the multi-source clone API) |
| Let a latest metadata read skip the GC boundary GET within a trust window | One GET less per poll of an unchanged manifest. | fork only |
| Add a flag that closes a database for writes | `ManifestV2.closed` (field 19), `Admin::close_for_writes`, `close_for_writes_at_epoch`, `reopen_for_writes_at_epoch`. | issue planned for the manifest field |
| Pin the vtable slots of `wal_name` and `closed` | A test that fails if a rebase moves the two fork fields. | fork only |
| Add a spawn hook that runs spawned tasks in a caller scope | Process-wide hook so an embedding application can scope SlateDB's background tasks (used for deterministic simulation and metrics). | fork only |
| Add `Db::wake_compactor` | Runs a compaction submitted with `Admin::submit_compaction` without waiting for the coordinator's poll. | fork only |
| Test that a retry of an applied final commit raises no signal | Regression test for the compactor's restart path. | fork only |
| Run one compaction job per destination | Two external submissions (`Admin::submit_compaction`) that name one destination were both promoted and claimed; the executor panicked and the database failed at every later open. Promotion, reclaim and the worker's claim now keep one job per destination. Plus a fix of a test that depended on the order of two ids made in one millisecond. | fork only |
| Record the final status of each finished compaction once | A job that finished after a job with a higher id left the `.compactions` ledger without any stored version saying how it ended; the next stored version now keeps it once. | fork only |

`git diff v0.18.0 <this branch>` lists the modified files.
