# Agent Notes

This repo contains a tiny Bash tool named `vsc`.

## Purpose

`vsc` starts or stops browser VS Code access for a repo using `code-server`.

It is for people who already manage their devbox, SSH, tmux, byobu, and coding agents themselves.

## Hard rules

- Do not manage tmux or byobu.
- Do not start, stop, attach to, or kill terminal sessions.
- Do not start or stop coding agents.
- Do not create a central project registry.
- Do not store secrets.
- Do not bind `code-server` to `0.0.0.0` by default.
- Do not modify a project's `.gitignore` automatically. Use `.git/info/exclude` for local ignores.
- Keep project-local editor state in `.code-server-web/`.
- Keep the tool small.

## Preferred behavior

- Prefer readable Bash over clever Bash.
- Prefer environment variables for optional overrides.
- Prefer stateless behavior where practical.
- Use deterministic ports derived from the repo path.
- Allow manual port override with `VSC_PORT`.
- Keep Tailscale Serve optional.
- Bind `code-server` to `127.0.0.1`.
- Use transient `systemd --user` services (`systemd-run --user`). There is no `.service` file on disk. The unit exists only while `code-server` is running and is gone after stop or crash.

## Non-goals

No web terminal. No project database. No workspace orchestration. No team management. No public internet exposure by default. No editor-state syncing between repos.

## Before changing behavior

```bash
bash -n vsc        # syntax check
shellcheck vsc     # lint (if available)
```

Core design: `vsc = code-server launcher only` — SSH, tmux, byobu, and agents are user-managed.
