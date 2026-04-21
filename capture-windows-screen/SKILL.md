---
name: capture-windows-screen
description: Capture the current Windows desktop from a WSL/OpenClaw environment and return a staged PNG path for inspection. Use when the user explicitly asks for a screenshot of the current Windows screen, desktop, or a visible app/window.
---

# Capture Windows Screen

Use the bundled script when the user explicitly asks for a Windows screenshot in this WSL-based environment. The script captures the screen through a configured Windows helper and prints a staged local PNG path.

## Quick workflow

1. Run `bash scripts/capture-windows-screen.sh`.
2. If you need a different local staging directory, set `STAGE_DIR=/path/to/staging` before running it.
3. Treat the printed path as the newest staged screenshot.
4. Use that image for inspection or any later user-approved step.

## Commands

Default staging:

```bash
bash scripts/capture-windows-screen.sh
```

Custom staging directory:

```bash
STAGE_DIR=/path/to/staging bash scripts/capture-windows-screen.sh
```

Expected output:

```text
/path/to/staging/latest-screen-YYYYMMDD-HHMMSS.png
```

## Environment overrides

If your machine uses different paths, override them with environment variables before running the script:

- `WIN_PS`
- `WIN_SCRIPT`
- `OUT_WIN`
- `OUT_WSL`
- `STAGE_DIR`
- `PS_EXEC_POLICY`

## Failure handling

- If the Windows helper cannot be launched, verify the configured PowerShell path and helper path.
- If the command succeeds but the PNG is missing, verify the configured output paths and rerun once.
- If your machine uses different host-side locations, prefer environment overrides instead of editing the workflow ad hoc.
- Do not invent alternate screenshot commands unless the configured path is clearly broken.
