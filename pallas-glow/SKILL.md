---
name: pallas-glow
description: >-
  Open a file in Glow as a read-only Herdr pane. Use when the user invokes
  /pallas-glow, asks to open a file in glow, or wants a rendered read-only
  markdown view while inside Herdr. Do not use when the agent only needs to
  read the file to answer a question, or when the user wants to edit the file.
---

# Open a file in Glow

Glow renders markdown in the terminal. It is read-only. Inside Herdr it runs in its own pane. Do not run `glow` in the agent pane. To edit a file, use `pallas-ttt`.

## Preflight

Stop unless this pane is Herdr-managed and the CLI works:

```bash
test "${HERDR_ENV:-}" = 1 && command -v herdr >/dev/null && herdr --version
```

If that fails, say you are not inside Herdr and stop.

## Glow

`command -v glow` must succeed. If it does not, Glow is not installed. Install it, then continue. If the install fails, report the failure and stop.

## Target

Use the file the user named. Resolve it to an absolute path. If they did not name one, ask. The path must exist before you split a pane. A directory is the working directory. A file uses its parent directory.

## Open

Inspect the caller pane. Split a wide pane to the right, or a narrow or tall pane down. Focus the new pane:

```bash
herdr pane layout --pane "$HERDR_PANE_ID"
herdr pane split --current --direction right --cwd "$CWD" --focus
```

Replace `right` with `down` when the caller pane is narrow or tall. `$CWD` is the directory from Target. Read the new pane ID from `.result.pane.pane_id`.

Start Glow's reader on a file. `-t` keeps the TUI open on that document:

```bash
herdr pane run "$PANE_ID" "glow -t $(printf '%q' "$TARGET")"
```

`$TARGET` is the absolute path. For a directory, drop `-t` so Glow opens its file browser there:

```bash
herdr pane run "$PANE_ID" "glow $(printf '%q' "$TARGET")"
```

If Glow reports that pager and tui cannot both be set, run the file with `glow -p` instead.

The new pane must be at a shell prompt. If the command arrives before the shell is ready, read the pane and run it again. Then confirm the document is on screen:

```bash
herdr pane read "$PANE_ID" --source visible --lines 40
```

Leave the pane open. Report the path you opened. Do not paste the file into the chat unless the user also asked for an explanation.
