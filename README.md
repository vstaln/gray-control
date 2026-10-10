<div align="center">
  <img alt="gray-control" src="assets/icon.svg" width="120" height="120" />
  <h1>gray-control</h1>
  <p><strong>Per-session remote control over a unix socket, using JSON-RPC.</strong></p>
  <p>
    <a href="https://gray.alignment.id">Website</a> ·
    <a href="https://gray.alignment.id/plugins/gray-control">Store</a> ·
    <a href="https://github.com/vstaln/gray-control">Source</a> ·
    <a href="https://github.com/vstaln/gray">gray</a>
  </p>
  <p>
    <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-1c1c20?style=flat-square&labelColor=0a0a0b" /></a>
    <a href="https://www.rust-lang.org"><img alt="Built with Rust" src="https://img.shields.io/badge/built%20with-rust-1c1c20?style=flat-square&labelColor=0a0a0b&logo=rust&logoColor=d4a373" /></a>
    <a href="https://gray.alignment.id/plugins/gray-control"><img alt="gray plugin" src="https://img.shields.io/badge/gray-plugin-1c1c20?style=flat-square&labelColor=0a0a0b&color=7aa2f7" /></a>
  </p>
</div>

<br/>

```bash
gray plugin install gray-control
```

Each gray session binds `~/.gray/control/<sid>.sock` (lazily, on the first
hook that names a session) and serves line-delimited JSON-RPC.

## RPC

One JSON object per line; replies echo `id` when the request has one.

```sh
echo '{"method":"send","text":"hi"}' | nc -U ~/.gray/control/<sid>.sock
```

| method | effect |
|---|---|
| `send` `{text}` | enqueues the text for the **next `agent/before_start`** — it lands as a user message (the reliable path). Also fires a best-effort `host/say` so it surfaces immediately when that capability is granted |
| `status` | `{ok, sid, cwd, uptime_s, sock, pending}` |
| `stop` | replies `{ok:true}`, then stops the listener and removes the socket |

## Tool

`send_to_session {sid, text}` — writes to another session's control socket.
Live session ids come from `/control list` (or globbing `~/.gray/control/`).

## Commands

- `/control` — socket path, listening state, `nc -U` usage
- `/control list` — live control sockets
- `/control stop` — stop this session's listener

The socket file is removed on `plugin/shutdown`.

## Wire

- claims: `agent/before_start` (drains the send queue into `{text}`),
  `tool/call` (`send_to_session`), `command/run` (`/control`),
  `plugin/manifest`, `plugin/shutdown`
- sidecar→host: `host/say` (fire-and-forget, optional)
- protocol `2.0`, capabilities: `host.say` (works degraded without it)

## Install

```
gray plugin install ~/grayplugins/gray-control
gray plugin capabilities control --all   # optional: enables host/say
```

## Tags

`gray` `plugin` `control` `rust`

---
Part of the [gray](https://github.com/vstaln/gray) plugin ecosystem —
the open-source AI agent harness. <https://gray.alignment.id>
