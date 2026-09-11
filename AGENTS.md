# AGENTS.md

## Cursor Cloud specific instructions

This repo is a Rust monorepo (Cargo workspace, ~80 crates) for **Grok Build**
(`grok`), a terminal AI coding agent. There is no web app, no database server,
and no docker-compose stack: state lives under `~/.grok/` with embedded SQLite.
Standard build/lint/test/run commands are documented in `README.md` (see the
"Building from source" and "Development" sections); the notes below only cover
non-obvious caveats.

### Build toolchain
- The Rust toolchain is pinned by `rust-toolchain.toml` (1.92.0) and is
  auto-installed by `rustup` on the first `cargo` command.
- Proto codegen (`build.rs` in several crates) needs `protoc`. It is resolved
  via the DotSlash wrapper `bin/protoc`, which requires `dotslash` on `PATH`.
  The startup update script runs `cargo install dotslash`; `bin/protoc` then
  downloads protoc 29.3 from GitHub on first build. If a build fails with
  "protoc not found / likely dotslash is missing", confirm `dotslash --help`
  works, or set `$PROTOC` to a system protoc.
- The root `Cargo.toml` is **generated — treat it as read-only**. Edit per-crate
  `Cargo.toml` files instead.
- Always target specific crates (`cargo build/check/test/clippy -p <crate>`);
  full-workspace builds are slow. The main binary is
  `cargo build -p xai-grok-pager-bin`, producing `target/debug/xai-grok-pager`
  (shipped to users as `grok`). A clean debug build of the binary takes several
  minutes.

### Running / testing without xAI credentials
- Real end-to-end use (interactive TUI `grok`, or headless `grok -p "..."`)
  needs xAI authentication and network access to `cli-chat-proxy.grok.com`.
  No credentials are provisioned in this environment by default.
- For credential-free end-to-end runs, tests spin up an in-process mock
  inference server (`xai-grok-test-support`) and point the real binary at it via
  `GROK_*_BASE_URL` + `XAI_API_KEY=test-key-for-ci`. The canonical headless
  E2E lives in `crates/codegen/xai-grok-shell/tests/test_built_binary_e2e.rs`.
- These built-binary E2E tests are marked `#[ignore]`; run them with
  `-- --ignored`, e.g.
  `cargo test -p xai-grok-shell --test test_built_binary_e2e -- --ignored`.
- Set `GROK_BINARY=$PWD/target/debug/xai-grok-pager` before running those tests
  to reuse an already-built binary and skip a rebuild.
