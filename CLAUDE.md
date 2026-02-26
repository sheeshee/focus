# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`focus` is a single-file bash script (`focus`) that toggles a "deep work mode" by:
- Appending `127.0.0.1` entries to `/etc/hosts` (between marker comments) to block distracting domains
- Flushing the macOS DNS cache via `dscacheutil` and `mDNSResponder`
- Optionally setting a Slack status via the Slack API (disabled by default, controlled by `SLACK_TOKEN`)
- Optionally running a background timer subprocess that notifies when a session expires

## Commands

```bash
focus start [minutes]   # Enter focus mode, optionally with a countdown timer
focus pause <minutes>   # Temporarily unblock sites for N minutes, then re-block
focus stop              # Exit focus mode
focus status            # Check current state and list blocked domains
```

## Configuration

User config lives at `~/.config/focus/config` (sourced as bash). The only meaningful variable is `SLACK_TOKEN` for Slack integration. `BLOCKED_DOMAINS` is edited directly in the script.

## Installation

```bash
ln -s $(pwd)/focus ~/.local/bin/focus
```

## Key Implementation Details

- Focus mode state is detected by grepping `/etc/hosts` for `# >>> FOCUS MODE START >>>` — no separate state file
- Paused state is detected by checking if the PID in `~/.config/focus/pause.pid` is still alive
- `sudo` is required for `/etc/hosts` writes and DNS flush commands
- The start timer is a detached background subshell; its PID is stored in `~/.config/focus/timer.pid`
- The pause timer is a detached background subshell; its PID is stored in `~/.config/focus/pause.pid`. It runs `sudo -v` before forking to pre-cache credentials, but pauses longer than the sudo timeout (~5 min) may prompt for a password when the timer fires
- Slack calls are commented out in `cmd_start`/`cmd_stop` by default and must be explicitly re-enabled
- macOS-only: DNS flush uses `dscacheutil` and `killall -HUP mDNSResponder`
