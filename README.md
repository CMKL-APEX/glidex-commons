# glidex-commons

Libraries that [glidex](https://github.com/CMKL-APEX/glidex) and
[hyperstor](https://github.com/CMKL-APEX/hyperstor) share: five Rust
crates, one helper binary, and the web UI kit. They moved here from
glidex, with their history, so that neither project builds from the
other's repository (hyperstor D356).

| Crate | What |
|---|---|
| `glidex-raftstore` | A replicated redb store on openraft: write sets recorded from transactions, applied as bytes on every member. |
| `glidex-tls` | HTTPS listeners on every address, self-signed certificates, the client trust shared by CLIs and UIs, Tailscale certificates. |
| `glidex-trust` | The cluster side of trust with another organization's glidex directory: offers, peer assertions, the feed, directory tokens; a fake directory for tests (`fake`). |
| `glidex-auth` | Secrets, session and token rules, a Cedar policy engine and OIDC login. |
| `glidex-authd` | The privileged PAM helper (binary and library): verifies local users' passwords for a service over a Unix socket. |
| `glidex-topology` | A host's topology from sysfs: NUMA nodes, the PCIe tree with root ports and switches, network ports with their RDMA devices. |
| `@glidex/ui-kit` (`ui-kit/`) | React pieces for sign-in, step-up and secrets shown once. |

The crates keep their `glidex-` names, so code using them didn't change
when they moved.

## Specs

The designs stay in glidex's `spec/`: where a crate's doc or description
cites `spec/NN-topic.md`, it means glidex's. Hyperstor's decisions about
these crates are in its own `spec/04-decisions.md`.

## Using it

Pin a release tag, the same in every crate of the workspace:

```toml
glidex-tls = { git = "https://github.com/CMKL-APEX/glidex-commons", tag = "v0.1.0" }
```

To try a change here from glidex or hyperstor before it is released,
point the dependents at a local checkout with a `[patch]` section, and
don't commit it:

```toml
[patch."https://github.com/CMKL-APEX/glidex-commons"]
glidex-tls = { path = "../glidex-commons/crates/glidex-tls" }
```

A binary from here, such as `glidex-authd`, is built from a dependent's
workspace with `cargo build -p glidex-authd`.

**The UI kit** is TypeScript source with no build step, and bun can't
install a package from a repository's subdirectory. Each app keeps an
unchanged copy at the tag it pins, replaced by its sync script and checked
against this repository in its CI. `react` and `@types/react` are peer
dependencies. The kit is type-checked from the repository's root
(`package.json`, `tsconfig.json`), so its own files are all an app copies.

## Releases

- Changes land on `main` through pull requests, with CI green.
- A release is a tag `vX.Y.Z` on `main`, with the workspace version set to
  it first. glidex and hyperstor then move their pins in their own pull
  requests.
- While the version is `0.x`, a minor bump (`0.1` → `0.2`) may break the
  API; a patch release doesn't.
- crates.io: the five library crates are published there (`glidex-authd`
  isn't: `publish = false`). The `crates.io` workflow packages each one on
  every change without uploading, lists what its package contains and
  fails if the license files are missing. An upload is a manual run from a
  release tag, and is permanent: a version can be yanked, never deleted or
  replaced.

### A release

1. Set `[workspace.package] version` and the `version` of
   `[workspace.dependencies] glidex-tls` to `X.Y.Z`, run the checks below,
   and merge.
2. Tag `vX.Y.Z` on `main` and make a GitHub release from it.
3. Actions → crates.io → Run workflow, from **the tag** (not `main`), with
   `publish` checked. The run refuses a `-dev` version, or a ref other
   than `refs/tags/vX.Y.Z`.
4. A reviewer of the `crates-io` environment approves the run, and it
   uploads.
5. Check the five crates on crates.io at the new version.
6. Move the pins in glidex and hyperstor, in their own pull requests.

The upload uses Trusted Publishing: crates.io accepts the workflow's OIDC
token, so the repository stores no crates.io token. The `crates-io`
environment has required reviewers, deploys only from `v*` tags and holds
no secrets.

### The first publish of a crate

Trusted Publishing can be set up only for a crate that already exists on
crates.io, so a crate's first version is uploaded by hand, by an owner
with their own crates.io account:

1. Make a crates.io API token with the `publish-new` scope only, limited
   to crates `glidex-*`, with a short expiry.
2. Publish from the release tag:

   ```sh
   git checkout vX.Y.Z
   cargo login          # the token at the prompt, never on a command line
   cargo publish --workspace --locked --dry-run
   cargo publish --workspace --locked
   cargo logout
   ```

   (`-p <crate>` instead of `--workspace` for one crate added later.)
3. Revoke the token on crates.io.
4. For each new crate, on crates.io: Settings → Trusted Publishing → add
   GitHub, with owner `CMKL-APEX`, repository `glidex-commons`, workflow
   `publish.yml`, environment `crates-io`.
5. Give each new crate a second owner
   (`cargo owner --add github:CMKL-APEX:<team> <crate>`), so that it
   doesn't depend on one account.

## Checks

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets --locked -- -D warnings
GLIDEX_CHECK_WRITE_SETS=1 cargo test --workspace --locked
RUSTDOCFLAGS=-D\ warnings cargo doc --workspace --no-deps --locked
bun install --frozen-lockfile && bun run typecheck   # the UI kit
```

## License

MIT OR Apache-2.0, at your option (`LICENSES/`).
