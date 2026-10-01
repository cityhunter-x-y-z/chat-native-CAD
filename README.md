# trimension

Agent-native 3D drafting canvas. Browser-native, ingests AutoCAD drawings, editable by a human and
an AI agent as peers on the same document.

- **Intent & build plan:** [`PRD.md`](./PRD.md)
- **Rules (invariants + anti-patterns):** [`CLAUDE.md`](./CLAUDE.md)
- **Pinned dependency versions:** [`VERSIONS.md`](./VERSIONS.md)
- **What was built and what is still open:** [`ARCHITECTURE.md`](./ARCHITECTURE.md)

## Status

**M0–M7 implemented.** 136 Rust tests, `clippy -D warnings` clean, all three compile
targets (browser wasm32, native server, native headless) building.

| Milestone | State |
| --- | --- |
| M0 scaffold | ✅ workspace, xtask, CI, three targets |
| M1 document + commit core | ✅ I1 is a compile error; hashing proven identical on wasm32 and native |
| M2 DXF import | ✅ four fixtures, data-driven rules, provenance on every classification |
| M3 renderer | ✅ 5k entities at 1920×1080 in 3.16 ms/frame including readback |
| M4 2D→3D | ✅ walls extrude with openings subtracted, exact volumes, closed meshes |
| M5 session server | ✅ two clients + a headless agent converge |
| M6 tool surface | ✅ five tools, schemas generated from Rust, drift guarded both ways |
| M7 chat-native canvas | ✅ four wgpu viewports in the browser, plan approval verified end to end |

## Views

Plan, left elevation, right elevation and a true two-point perspective, all rendered by
the same `wgpu` code that produces an agent's headless PNG. Click a wall in the plan to see
where each of its numbers came from; drag the perspective to orbit; use ⤢ to maximise a
viewport.

Known findings — including two decisions a drafter would dispute that are not yet fully
traceable — are listed in ARCHITECTURE.md §1 and §2.

## Prerequisites

- **Rust** via [rustup](https://rustup.rs). `rust-toolchain.toml` pins 1.98.1, so rustup
  fetches the right version on first `cargo` run. Then add the browser target:
  `rustup target add wasm32-unknown-unknown`.
- **wasm-bindgen CLI**, the exact version of the `wasm-bindgen` crate in `Cargo.lock` (a
  mismatch fails at link time): `cargo install wasm-bindgen-cli --version 0.2.128`.
- **Node >= 22.6** (the agent runs TypeScript with `--experimental-strip-types`).

No toolchain ships in the repo. If you would rather keep Rust out of `~/.cargo`, point
`RUSTUP_HOME` and `CARGO_HOME` at a local `.toolchain/` (gitignored) before installing:

```bash
export RUSTUP_HOME="$PWD/.toolchain/rustup" CARGO_HOME="$PWD/.toolchain/cargo"
export PATH="$PWD/.toolchain/cargo/bin:$PATH"
```

## Running it

The Wasm bundle is a build artifact and is not checked in; `npm run wasm` builds it.

```bash
cd apps/web && npm install && npm run wasm && npm run dev   # shell on http://localhost:6178
cargo run -p tri-server                                      # from the repo root; session server on :8788
```

The agent is a one-shot CLI that talks to the session server:

```bash
cd apps/agent && npm install
ANTHROPIC_API_KEY=... npm start -- "<what you want done>"
```

Tests:

```bash
cargo xtask test-all
```

## Ports

| Service | Port |
| ------- | ---- |
| web shell | 6178 |
| session server | 8788 |
