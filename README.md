<p align="center">
  <img src="assets/gray-logo.svg" alt="gray" width="96">
</p>
<h1 align="center">gray-control</h1>
<p align="center">Remote-control a running gray session over a per-session unix socket.</p>
<p align="center">
  <a href="https://github.com/vstaln/gray-control/blob/main/LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-blue.svg"></a>
  <img alt="gray plugin" src="https://img.shields.io/badge/gray-plugin-7aa2f7.svg">
  <img alt="rust" src="https://img.shields.io/badge/built%20with-rust-orange.svg">
</p>

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

---
Part of the [gray](https://github.com/vstaln/gray) plugin ecosystem —
the open-source AI agent harness. <https://gray.alignment.id>
