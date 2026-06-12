# vsc

Tiny launcher for `code-server` — opens a repo in browser VS Code.

Manages only `code-server`. Does not touch tmux, byobu, agents, or project lifecycle.

## Typical workflow

1. SSH into your devbox.
2. Attach to tmux/byobu.
3. `cd` into your repo.
4. `vsc up` → get a browser VS Code URL.
5. Open the URL from any device on your private network (e.g. Tailscale).

## Dependencies

Required: `code-server`, `systemd-run`, `python3`, `ss`

Optional: `tailscale` (only used to print the tailnet IP in the URL)

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
```

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
