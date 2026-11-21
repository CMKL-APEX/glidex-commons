# Claude Code in this repository

@AGENTS.md

- Use your own `CARGO_TARGET_DIR`, never another checkout's.
- On shared development hosts, build with `-j1` or `-j2`.
- Long runs (workspace tests, clippy) go in the background.
- Commits and PRs end with the attribution lines your harness gives you.
