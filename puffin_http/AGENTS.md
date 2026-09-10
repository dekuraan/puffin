# puffin_http

Upstream: <https://github.com/EmbarkStudios/puffin>. TCP server/client that
streams `puffin` frame data so an external `puffin_viewer` can attach live.
Native-only — uses `std::net::TcpListener`, which doesn't compile for
`wasm32-unknown-unknown`.

## How wam uses it

`wam-launcher/src/native.rs::start_puffin_http` (around line 303) starts the
server when the launcher is built with `--features profile-with-puffin`:

```rust
let addr = format!("0.0.0.0:{}", puffin_http::DEFAULT_PORT);   // 8585
puffin_http::Server::new(&addr)
```

`wam-launcher/Cargo.toml` declares the crate optional + native-only:

```toml
puffin_http = { version = "0.16", optional = true }   # native target only
profile-with-puffin = ["...", "dep:puffin_http", ...]
```

The viewer is invoked separately as a standalone binary:

```sh
puffin_viewer --url 0.0.0.0:8585
```

The WASM build never imports this crate — the F9 hotkey writes a `.puffin`
file via the patched `puffin::FrameView::write` (see `puffin/AGENTS.md`)
and the user opens it in `puffin_viewer` after the fact.

## When changing this crate

- Treat it as native-only forever. If you find yourself adding `cfg`-gated
  wasm support, redesign — the wam workflow is "save .puffin, open later"
  on wasm by design.
- Bumping the puffin core version usually requires a matching `puffin_http`
  bump because they share the on-the-wire frame format.
