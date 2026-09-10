# puffin_viewer

Upstream: <https://github.com/EmbarkStudios/puffin>. Standalone egui app for
inspecting `puffin` data — connects to a live `puffin_http` server (`--url`)
or opens a saved `.puffin` file (positional arg).

## wam does not depend on this crate

`puffin_viewer` is used as an installed external binary, not as a library
dep. The recommended install path is still upstream:

```sh
cargo install puffin_viewer
puffin_viewer --url 0.0.0.0:8585         # attach to a running native client
puffin_viewer profile-<unix>.puffin       # open a saved file (incl. wasm F9)
```

Nothing in the WAM workspace links against `puffin_viewer`. The crate sits in
this submodule because it's part of the upstream workspace; we don't ship it
or bundle it.

If you need to cross-check a saved frame programmatically, use the `puffin`
crate's `FrameView::read` directly from one of the
`crates/wam-launcher/examples/puffin_*.rs` analysis scripts — they already
do that and are the supported path.

## When changing this crate

Don't, unless the change is going upstream. We carry no patches here.
