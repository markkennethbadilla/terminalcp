# 001: terminalcp works on Windows

## What it does

This fork of Mario Zechner's terminalcp runs on Windows. terminalcp is an MCP
server that starts named terminal sessions, sends them keystrokes and reads
their screen.

## Why it exists

Upstream (badlogic/terminalcp, last push 2025-08-17) breaks on Windows in two
places, and nothing else does the job as well:

- It listens on the Unix socket `~/.terminalcp/server.sock`. Windows Node
  cannot listen there, so every session start fails with `listen EACCES` or
  `Failed to start server` while the MCP handshake still looks healthy.
- It finds its own script with `new URL(import.meta.url).pathname`, which is
  `/C:/...` on Windows and cannot be spawned.

Until this fork, mkb-agentops patched the installed files after every npm
install. Owning the fork removes that patch step.

## How it works

- On Windows the server and clients use the named pipe
  `\\.\pipe\terminalcp-<username>`; elsewhere the upstream socket path is
  unchanged. The expression sits in `src/attach-client.ts`, `src/index.ts`,
  `src/terminal-client.ts` and `src/terminal-server.ts`.
- `src/terminal-server.ts` uses `fileURLToPath(import.meta.url)`.
- `npm run build` and `prepare` are plain `tsc`, so `npm install` from a git
  tag builds `dist/`. husky is removed. `node-pty` is 1.1.0, which ships
  Windows prebuilds, so no compiler is needed.
- The version is `1.3.3-mkb.N`. Each release is a git tag `v1.3.3-mkb.N`.

## What it reads and writes

The same as upstream: it spawns the commands it is asked to run, holds their
output in memory, and listens on the named pipe (Windows) or
`~/.terminalcp/server.sock`. It writes no other files.

## How to run, check, and hand over

- Build: `npm ci` then `npm run build`.
- Check: CI on every pull request and push to `main` runs the build on
  Windows and fails if `dist/terminal-server.js` lacks the named pipe.
  Expected CI use is about 20 minutes a month.
- Consumer: mkb-agentops installs it with
  `npm install --prefix %USERPROFILE%\terminalcp github:markkennethbadilla/terminalcp#<tag>`
  (`versions.json`, entry `terminalcp`). To release, bump `version`, tag
  `v<version>`, push the tag, then change the tag in that entry.
- Upstream changes: `git fetch upstream` (remote `badlogic/terminalcp`) and
  merge.
