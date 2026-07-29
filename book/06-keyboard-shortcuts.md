# Keyboard shortcuts

_II · Controls · Chapter 06_

If you memorise five keys, memorise these: `Esc` interrupts, `Esc Esc` rewinds, `Ctrl+O` shows what really happened, `Shift+Tab` changes how much Claude asks, and `Ctrl+G` opens your prompt in a real editor. Everything below is the long form.

## When things are going wrong

| Key | Behaviour |
| --- | --- |
| `Esc` | Interrupts Claude mid-response or mid-tool-call so you can redirect. **Work done so far is kept.** If a dialog is open, `Esc` closes the dialog instead. |
| `Esc` `Esc` | Input has text → clears the draft (saved to history; press `Up` to recall). Input empty → opens the **rewind menu**. |
| `Ctrl+C` | Interrupts a running operation. If nothing is running: first press clears input, second press exits. |
| `Ctrl+D` | Exit. First press shows a confirmation hint; a second within 800 ms exits. With text in the prompt, deletes the character after the cursor instead. |
| `Ctrl+X Ctrl+K` | Stop **all** running background subagents in this session. Press twice within 3 seconds to confirm. |
| `Ctrl+L` | Redraw the screen. Recovers a garbled or half-blank terminal without losing history. |
| `Ctrl+Z` | Suspend to your shell (Unix only). `fg` resumes. |

## Session and mode control

| Key | Behaviour |
| --- | --- |
| `Shift+Tab` | Cycle permission modes: `default` → `acceptEdits` → `plan`, plus any you have enabled. On Windows without VT input mode, `Alt+M`. |
| `Ctrl+O` | Toggle the **transcript viewer**: detailed tool usage, timestamps, the model used per message, and expanded MCP calls that otherwise collapse to “Called slack 3 times”. |
| `Ctrl+T` | Toggle Claude’s to-do checklist in the status area. (Inside the `/theme` picker, `Ctrl+T` toggles syntax highlighting instead.) |
| `Ctrl+B` | Background running tasks — Bash commands and agents. tmux users press twice. |
| `Ctrl+R` | Reverse-search input history. |
| `Ctrl+S` | Stash the current prompt and clear it. Press again on an empty prompt to restore text, cursor position, and pasted content. |
| `Ctrl+G` / `Ctrl+X Ctrl+E` | Open the prompt — or the proposed plan — in `$EDITOR`. **The single most underused key in Claude Code.** |
| `Option+P` / `Alt+P` | Switch model without clearing your prompt. |
| `Option+T` / `Alt+T` | Toggle extended thinking. No effect on Fable 5, which always thinks. |
| `Option+O` / `Alt+O` | Toggle fast mode. → Ch. 28 |
| `Ctrl+V` / `Cmd+V` / `Alt+V` | Paste an image from the clipboard. Inserts an `[Image #N]` chip you can reference positionally. |
| `?` on empty input | Toggle the shortcut help panel. |

## Line editing (readline bindings)

| Key | Behaviour |
| --- | --- |
| `Ctrl+A` / `Ctrl+E` | Start / end of the current logical line |
| `Ctrl+K` / `Ctrl+U` | Delete to end of line / to line start — both store the text for pasting |
| `Ctrl+W` | Delete previous word (also `Option+Delete` on macOS, `Ctrl+Backspace` on Windows) |
| `Ctrl+Y`, then `Alt+Y` | Paste deleted text; `Alt+Y` cycles the paste history |
| `Alt+B` / `Alt+F` | Move back / forward one word |
| `Ctrl+_` | Undo the last input edit, restoring text and cursor position |
| `Up` / `Down` | Move the cursor within a multi-row prompt; once on the first or last row, navigate command history |

> **Note — macOS: enable Option as Meta first**
>
> `Alt+B`, `Alt+F`, `Alt+Y`, and `Alt+P` need the Option key mapped to Meta. **iTerm2:** Settings → Profiles → Keys → set Left/Right Option to “Esc+”. **Apple Terminal:** Settings → Profiles → Keyboard → “Use Option as Meta Key”. **VS Code:** set `"terminal.integrated.macOptionIsMeta": true`.

## Multiline input — five ways, pick one

| Method | Where it works |
| --- | --- |
| `\` then `Enter` | **Everywhere.** The portable answer; use it if you only learn one. |
| `Ctrl+J` | Any terminal, no configuration |
| `Shift+Enter` | Native in iTerm2, WezTerm, Ghostty, Kitty, Warp, Apple Terminal, Windows Terminal. For VS Code, Cursor, Alacritty, and Zed run `/terminal-setup` to install the binding. |
| `Option+Enter` | macOS, after enabling Option as Meta |
| Paste directly | Code blocks and logs paste as-is |

## Inside the transcript viewer (`Ctrl+O`)

| Key | Behaviour |
| --- | --- |
| `{` / `}` | Jump to previous / next user prompt, vim-paragraph style (fullscreen rendering) |
| `Ctrl+E` | Toggle “show all content” (default renderer only) |
| `[` | Write the whole conversation to native terminal scrollback so `Cmd+F` and tmux copy-mode can search it |
| `v` | Dump the conversation to a temp file and open it in `$VISUAL`/`$EDITOR` |
| `?` | Show the full shortcut reference panel |
| `q` / `Ctrl+C` / `Esc` | Exit transcript view |

> **Key — Vim mode exists**
>
> Set `editorMode: "vim"` in settings, or `/config` → **Editor mode**. The old `/vim` toggle was removed in v2.1.92. You get NORMAL and INSERT modes with the expected navigation (`h j k l`, `w b e`, `0 $ ^`, `gg G`), editing (`x d c y p`, `dd`, `D C`, `. u`), text objects (`iw`, `aw`, `i"`, `ib`), and visual mode. INSERT-mode key sequences are remappable. Worth it only if vim is already muscle memory.
