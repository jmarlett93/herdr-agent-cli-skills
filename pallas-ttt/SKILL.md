---
name: pallas-ttt
description: >-
  Open a source file in TTT Editor as a Herdr pane. Use when the user invokes
  /pallas-ttt, asks to open a file in ttt, or asks to view or edit code in the
  terminal editor while inside Herdr. Do not use when the agent only needs to
  read a file to answer a question.
---

# Open a file in TTT

TTT is the terminal IDE. Inside Herdr it runs in its own pane via the `ttt.editor` plugin. Do not launch `ttt` in the agent pane. For a rendered read-only markdown view, use `pallas-glow`.

## Preflight

Stop unless this pane is Herdr-managed and the CLI works:

```bash
test "${HERDR_ENV:-}" = 1 && command -v herdr >/dev/null && herdr --version
```

If that fails, say you are not inside Herdr and stop.

## Target

Use the file or directory the user named. Resolve it to an absolute path. If they did not name one, ask. A directory opens that workspace. A file opens that file.

## Plugin

`herdr plugin list` must show `ttt.editor`. If it does not, install it, then continue:

```bash
herdr plugin install eugenioenko/ttt/herdr-plugin
```

## Open

The plugin reads `TTT_TARGET_DIR`. A directory becomes the workspace. A file is passed to `ttt` as that file.

```bash
herdr plugin pane open \
  --plugin ttt.editor \
  --entrypoint editor \
  --env "TTT_TARGET_DIR=$TARGET" \
  --focus
```

`$TARGET` is the absolute path. Focus the new pane so the user lands in the editor. Report the path you opened. Do not paste the file into the chat unless the user also asked for an explanation.
