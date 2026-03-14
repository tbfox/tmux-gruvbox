# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make lint_shellcheck    # Run shellcheck on all .sh files
make lint_shfmt         # Run shfmt formatter check on all .sh files
make check_scripts      # Run both linters

# Tests (Linux only — requires tmux, git, bash)
bash tests/run_all_linux_tests.sh
```

## Architecture

This is a tmux theme plugin for the [Gruvbox](https://github.com/morhetz/gruvbox) colorscheme, designed to be installed via [TPM](https://github.com/tmux-plugins/tpm).

**Entry point:** `gruvbox-tpm.tmux` sources `src/gruvbox-main.sh` when TPM loads the plugin.

**`src/gruvbox-main.sh`** is the orchestrator:
1. Reads user config options from tmux (`@tmux-gruvbox`, `@tmux-gruvbox-statusbar-alpha`, status section overrides)
2. Sources the matching palette file (colors) and theme file (tmux option assignments)
3. Builds an array of tmux `set` commands and executes them all at once

**4 theme variants**, each with a palette + theme file pair:
- `dark` / `light` — 16-bit hex colors (e.g., `#282828`)
- `dark256` / `light256` — 256-color xterm codes (e.g., `colour235`)
- Default fallback is `dark256`

**File roles:**
- `src/palette_gruvbox_*.sh` — Define color variables (`col_bg`, `col_fg`, `col_red`, etc.)
- `src/theme_gruvbox_*.sh` — Consume palette variables to build tmux option strings (status bar, panes, windows, messages)
- `src/tmux_utils.sh` — Helpers to read/write tmux options (`tmux_get`, `tmux_set`, `append_tmux_option`)
- `src/helper_methods.sh` — Debug utility (`print_array`)

**Status bar layout:** Left section A (default: session name), right sections X/Y/Z (default: date/time/hostname). Each is configurable via tmux options. Transparency (`@tmux-gruvbox-statusbar-alpha`) sets the statusbar background to `default` instead of a solid color; requires tmux 3.2+.

## Code Style

Shell scripts follow POSIX conventions. Enforced by:
- `shellcheck` for linting
- `shfmt` for formatting (2-space indent, as per `.editorconfig`)
- EditorConfig: UTF-8, LF line endings

## Tests

Integration tests in `tests/linux/` spin up a real tmux session with a minimal `.tmux.conf`, install the plugin via TPM, then assert on `tmux show-options` output. There are 9 test cases covering all 4 theme variants, with and without transparency, plus the fallback behavior. Tests only run on Linux.
