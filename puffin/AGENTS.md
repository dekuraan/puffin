# puffin (core)

Upstream: <https://github.com/EmbarkStudios/puffin>. Frame-data store, scope
collection, packing/serialization, `GlobalProfiler`. Pure-Rust, no GUI.

## Why wam vendors this crate

Submoduled (not crates.io) so we can carry one local patch on top of
`puffin-0.19.1`:

- **`src/frame_data.rs`** — drops `#[cfg(not(target_arch = "wasm32"))]` from
  `FrameDataState::packed`, `FrameDataState::pack_and_keep`,
  `FrameView::create_packed`, and `FrameView::write_into`.
- **`src/profile_view.rs`** — drops the same gate from `FrameView::write`.

Upstream gates these off on `wasm32` with the comment
"compression not supported on wasm". `PackedStreams::pack` only uses
`lz4_flex` though — pure-Rust and wasm-safe — so the gate is spurious.
Removing it lets the F9 hotkey in `wam-launcher::wasm::install_profile_hotkey`
write a downloadable `.puffin` blob from the browser.

When rebasing onto a newer puffin tag, re-apply the `wam-fork` commit
(`6f545b4` at first cut). The five edits all sit on the same form
(`#[cfg(not(target_arch = "wasm32"))] // compression not supported on wasm`)
and should rebase cleanly unless upstream restructures `FrameView`.

## Features wam turns on

In `wam-launcher/Cargo.toml`:

```toml
puffin = { workspace = true, features = ["serialization"], optional = true }
```

Pulled in by the `profile-with-puffin` cascade. `serialization` activates the
`packing` → `lz4` chain — the same code path the patch keeps alive on wasm.
WASM additionally needs `web` for `js-sys` / `web-time`; that's gated on the
launcher side via `cfg(target_arch = "wasm32")`, not here.

## When changing this crate

- Per-frame allocations and lock contention show up in flamegraphs
  immediately; the WAM project measures puffin overhead via its own
  `profile-with-puffin` builds (see root `AGENTS.md` §Profiling).
- The instrumentation-vs-zero-cost story relies on the `profiling` crate
  compiling away when no `profile-with-*` feature is on. Don't add
  unconditional puffin calls from anywhere downstream — always go through
  `profiling::scope!` / `profiling::function`.
- If you touch packing/compression code, rerun the WASM F9 export to confirm
  the .puffin file still round-trips through `puffin_viewer`.
