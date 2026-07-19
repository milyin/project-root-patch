# workspace-root-patch

A tiny utility crate that, when built inside a Cargo workspace, exposes that
workspace's root to build-time consumers.

## `project-root` vs `workspace_root`

This package uses two deliberately different crates:

- [`project-root`](https://crates.io/crates/project-root) is an upstream,
  build-only dependency. Its `project_root::get_project_root()` call discovers
  the root of the workspace containing the helper crate itself.
- `workspace_root` is this package's public Rust library. Its
  `workspace_root::get_workspace_root()` function returns the root of the
  destination workspace after this helper has been patched into that workspace.

The distinction matters because a normal crates.io dependency is built under
Cargo's cache rather than the destination workspace. `workspace-root-patch`
copies itself into the destination workspace and adds a Cargo patch, making the
`workspace_root` library report the right location.

The main purpose of this tool is to allow a crate installed from crates.io (i.e., not part of the workspace)
to find the workspace's `Cargo.lock` at compile time. A direct
`project_root::get_project_root()` call from a normal dependency does not solve
this problem: during a build it would discover Cargo's cache rather than your
destination workspace.

This crate solves the problem by injecting itself into the user's workspace via the `[patch]` section.
That is, the workspace contains a copy of the `workspace-root-patch` crate which, being inside the
workspace, can determine the workspace root. Any crate that depends on `workspace-root-patch` will then
use the copy from the user's workspace instead of the one in `$HOME/.cargo`.

The Cargo subcommand `workspace-root-patch` automates this injection. Run:

```sh
cargo install workspace-root-patch
cargo workspace-root-patch install .
```

Build scripts can then use:

```rust
let root = workspace_root::get_workspace_root();
```
