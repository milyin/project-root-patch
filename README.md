# project-root-patch

A tiny utility crate that, when built inside a Cargo workspace, discovers the workspace root.

## Relation to `project-root`

[`project-root`](https://crates.io/crates/project-root) is the upstream crate
used by this package's `build.rs`. It can discover the workspace that contains
the crate currently being built.

`project-root-patch` solves the additional placement problem: an ordinary
crates.io dependency is built from Cargo's cache, so `project-root` would find
the cache rather than the consuming project's workspace. The
`cargo project-root-patch install` command copies this helper into the
destination workspace and adds a Cargo patch. Once built from that local copy,
the helper uses `project-root` to discover—and exposes—the destination
workspace root to its dependents.

The main purpose of this tool is to allow a crate installed from crates.io (i.e., not part of the workspace)
to find the workspace's `Cargo.lock` at compile time. A simple call to the well-known
`project_root::get_project_root()` doesn't work here because it searches upward from the current working
directory, which during a build is inside `$HOME/.cargo` rather than your workspace.

This crate solves the problem by injecting itself into the user's workspace via the `[patch]` section.
That is, the workspace contains a copy of the `project-root-patch` crate which, being inside the
workspace, can determine the workspace root. Any crate that depends on `project-root-patch` will then
use the copy from the user's workspace instead of the one in `$HOME/.cargo`.

The Cargo subcommand `project-root-patch` automates this injection. Run:

```sh
cargo install project-root-patch
cargo project-root-patch install .
```
