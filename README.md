# vsc

Tiny launcher for `code-server` — opens a repo in browser VS Code.

Manages only `code-server`. Does not touch tmux, byobu, agents, or project lifecycle.

## Typical workflow

1. SSH into your devbox.
2. Attach to tmux/byobu.
3. `cd` into your repo.
4. `vsc up` → get a browser VS Code URL.
5. Open the URL from any device on your private network (e.g. Tailscale).
   Tailnet devices get an `https://` URL; LAN devices use plain `http://ip:port`.

## Dependencies

Required: `code-server`, `systemd-run`, `python3`, `ss`, `curl`

Optional: `tailscale` (tailnet URLs + https via `tailscale serve`)

## Install

```bash
chmod +x vsc
mkdir -p ~/.local/bin
ln -sf "$PWD/vsc" ~/.local/bin/vsc
```

Add `~/.local/bin` to your `PATH` if needed.

## Usage

```bash
vsc up              # start code-server for current repo
vsc up ~/src/proj   # start for a specific repo
vsc down            # stop current repo's code-server
vsc status          # show active vsc services and their URLs
```

Environment overrides:

```bash
VSC_PORT=9123 vsc up                           # specific port
VSC_BASE_PORT=9100 VSC_PORT_RANGE=800 vsc up   # change port range
VSC_HTTPS_OFFSET=1000 vsc up                   # tailscale https port = port + offset
```

## HTTPS over Tailscale

When `tailscale` is running, `vsc up` also configures `tailscale serve` so the
instance is reachable at `https://<machine>.<tailnet>.ts.net:<port+1000>` with
a real Let's Encrypt certificate. Plain `http://ip:port` access (LAN and
tailnet) keeps working unchanged.

Details:

- The https port is `port + 1000` (override with `VSC_HTTPS_OFFSET`). It must
  differ from code-server's port: `tailscale serve` binds real sockets on the
  tailscale IPs and races a `0.0.0.0` bind on the same port.
- `tailscale serve` needs permissions. Either run
  `sudo tailscale set --operator=$USER` once, or have passwordless sudo; if
  neither works, `vsc up` prints a warning and http access still works.
- The certificate is per machine (not per port). It is provisioned lazily on
  the first https request — that one request may take up to a minute; all
  later requests (any port) are instant. Requires MagicDNS + HTTPS
  certificates enabled in the tailnet admin console.
- `vsc down` removes the serve entry. Serve config persists across reboots
  (`--bg`), so after a reboot a stale entry may linger until the next
  `vsc up`/`vsc down` for that repo — harmless, visible via
  `tailscale serve status`.

## State

| What | Where |
|------|-------|
| Editor state | `<repo>/.code-server-web/user-data/` |
| Extensions | `~/.local/share/code-server/shared-extensions/` |
| Systemd unit | `vsc-<port>-<name>.service` (transient — no file on disk) |

`.code-server-web/` is added to `.git/info/exclude` automatically.

The systemd unit is created with `systemd-run --user` — it lives only in memory while `code-server` is running. There is no `.service` file on disk. The unit is gone after `vsc down` or if `code-server` exits.

## Security

`code-server` runs `--auth none --bind-addr 0.0.0.0:<port>` — no authentication, listening on all interfaces. This is only acceptable when the host is reachable solely over a private network such as a tailnet. Do not run this on a host with a public IP and open firewall.

The `tailscale serve` https endpoint is tailnet-only (it is not Funnel — nothing is exposed to the public internet). It adds transport encryption, not authentication: tailnet ACLs remain the access-control boundary.
