# Notes for coding agents

Shared libraries of glidex and hyperstor (README.md). Both projects pin a
release tag of this repository, so a change here reaches them only when
they move their pins.

- **Public repository.** Never commit internal hostnames, addresses,
  serial numbers, credentials or other secrets, in code, tests, docs or
  logs.
- **Keep the crates independent of glidex and hyperstor.** Nothing here
  depends on either project's other crates; a type one project alone needs
  belongs in that project.
- **Compatibility.** A change that breaks a public API or a stored format
  (raftstore's snapshots and write sets, auth's tokens) says so in its
  pull request, and comes with the dependents' pull requests that adapt
  to it. Stored formats stay readable by the release before.
- **Designs** are in glidex's `spec/`; a design change is decided there
  (or in hyperstor's `spec/04-decisions.md` for what only hyperstor
  needs), then built here.
- **Style:** Rust 2024, rustfmt defaults; every `unsafe` block has a
  `// SAFETY:` comment; new behaviour comes with tests, and anything that
  persists data or crosses a crash gets a randomized test against a model.
- **Before a pull request**, run the checks in README.md, "Checks"; CI
  runs them all.
- Work on a branch in your own worktree; open pull requests against
  `main`. Commit and push only when asked.
- License: MIT OR Apache-2.0.
