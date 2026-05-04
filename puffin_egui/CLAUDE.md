# puffin_egui

Upstream: <https://github.com/EmbarkStudios/puffin>. In-process flamegraph /
stats UI built on egui. Useful when an app already runs egui and wants the
profiler view embedded in the same window.

## wam does not depend on this crate

WAM uses Bevy UI + custom dev panels (`wam-client::system_timings`,
`wam-client::dev_panels`) for the in-game perf overlays, and exports
`.puffin` files for offline analysis in the standalone `puffin_viewer`. There
is no egui in the workspace, and the standing decision is to keep it out
([feedback_ui_bevy_not_egui.md] in user memory).

The crate is built only because it sits in the upstream workspace; nothing
in the WAM `Cargo.toml`s depends on `puffin_egui`. Don't add a dependency on
it from a wam crate without checking with the user first — pulling in egui
just for a debug overlay is the case the memory note rules out.

## When changing this crate

Don't, unless the change is going upstream. We carry no patches here.
