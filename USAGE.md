# Dev environment — usage guide

Your setup, and how to get the most out of it. Everything here is consistent across
your Macs and Linux boxes (managed by chezmoi). Colors are colorblind-safe (protanopia)
everywhere via one shared palette.

---

## The daily workflow, in one picture

```
Ghostty  ──►  tmux (C-a)  ──►  { nvim | shell | claude }  ──►  review with delta / hunk
   │              │                                                     │
 OSC52         persistent                                         git review (hunk TUI)
 clipboard     sessions                                          delta pager for diffs
```

**Primary loop:** Claude Code writes code → you review the changeset in the terminal
with **`git review`** (hunk) or **delta** → stage with **lazygit** → commit.

---

## Terminal — Ghostty

| Action | How |
|---|---|
| Reload config | `⌘⇧,` (after editing `~/.config/ghostty/config`) |
| Copy from remote | Just select — OSC 52 pushes it to the Mac clipboard through SSH+tmux |
| Multiline in Claude/shell | **Shift+Enter** (bound to a real newline) |
| New tab / split | `⌘T` / `⌘D` (native — you can use these instead of tmux locally) |
| List color themes | `ghostty +list-themes` |

Colors use an Okabe-Ito palette: "red"→vermillion, "green"→teal, so error/success are
separable without the red/green axis.

---

## Multiplexer — tmux (prefix `C-a`)

| Action | Keys |
|---|---|
| Prefix | `Ctrl-a` (screen-style) |
| Split | `C-a "` (horizontal), `C-a %` (vertical) |
| Move panes | `C-a h/j/k/l` |
| Copy mode | `C-a [` then select (`v`), yank (`y`) → system clipboard |
| New / next window | `C-a c` / `C-a n` |
| Detach (leave running) | `C-a d` |
| Reload config | `C-a :source-file ~/.config/tmux/tmux.conf` |

Sessions survive SSH drops; `resurrect`/`continuum` persist them across reboots.
Status bar: **blue** = normal, **gold** = active window (CVD-safe, not the default green).

---

## SSH into a remote box

With an SSH host alias (in `~/.ssh/config` — not this repo): one alias attaches your
tmux session directly (auto-creating if none), another gives a plain shell. Connections
multiplex (instant reconnects) and keep-alive through laptop sleep. Your Mac SSH key is
forwarded, so `git push` from the remote uses it. (Run once on the Mac:
`ssh-add --apple-use-keychain ~/.ssh/id_ed25519`.)

---

## Shell (zsh on Mac, bash on Linux)

| Tool | What you get | Try |
|---|---|---|
| **zoxide** | Jump to frecent dirs | `z <partial-dir>` |
| **autojump** | classic dir jump | `j <partial-dir>` |
| **atuin** | Full-text, synced shell history | `Ctrl-r` |
| **fzf** | Fuzzy-find anything | `Ctrl-t` (files), `Ctrl-r` (history) |
| **direnv** | Per-directory env auto-load | `direnv allow` in a dir |
| **eza** | Modern `ls` | `ll`, `lt` (tree) |
| **bat** | Syntax-highlighted `cat` | `cat file.py` |

A once-a-day check on shell start warns if your dotfiles are behind (`chezmoi update`).

---

## Code review (the habit to build)

You review AI/dev changes without leaving the terminal.

| Tool | Use it for | Command |
|---|---|---|
| **delta** | Everyday diffs (auto pager) | `git diff`, `git show`, `git log -p` |
| **hunk** | Deep review of a changeset (multi-file sidebar, agent annotations) | `git review` (working tree), `git reviewlast` (last commit), `hunk diff --watch` |
| **difftastic** | Structural, syntax-aware diff | `difft a.py b.py` |
| **lazygit** | Stage/unstage **hunks**, commit, branch | `lazygit` |
| **tig** | Browse history | `tig` |
| **gh** | PR review from terminal | `gh pr view`, `gh pr diff` |

**Reviewing a Claude Code change:** after Claude edits files, run **`git review`** to
open hunk and page through the changeset (agent annotations show inline), then stage
the hunks you approve (`lazygit` or `git add -p`) and commit.

---

## Neovim (`nvim`) — leader is `Space`

Kickstart-style: LSP, Treesitter, fuzzy find, git signs — all CVD-themed.

| Action | Keys |
|---|---|
| Find files / grep / buffers | `<Space>ff` / `<Space>fg` / `<Space>fb` |
| Diagnostics list | `<Space>fd` |
| Goto definition / references | `gd` / `gr` |
| Hover docs | `K` |
| Rename / code action | `<Space>rn` / `<Space>ca` |
| Plugin manager | `:Lazy` |
| LSP servers | `:Mason` |

Diagnostics: error=vermillion, warn=gold, info=blue, hint=cyan — no red/green reliance.
Yanks go to the system clipboard (OSC 52 works even on a headless Linux box).

---

## Managing the setup — chezmoi

| Command | Does |
|---|---|
| `chezmoi edit ~/.zshrc` | Edit a managed file's source |
| `chezmoi apply` | Write changes into `$HOME` |
| `chezmoi update` | `git pull` + apply (sync a machine) |
| `chezmoi status` | Show local drift |
| `chezmoi cd` | Open the source repo |
| `chezmoi re-add` | Pull live edits back into the source |

**New machine:** `sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply tonyferrell`

**Change a color everywhere:** edit `.chezmoidata/colors.yaml`, `chezmoi apply` — Ghostty,
tmux, git-delta, and nvim all update from that one file.

### Secrets & personal/work layering
Each machine picks `personal`/`work` once at `chezmoi init`. Templates gate on
`.isWork`/`.isPersonal`. Secrets come from **Bitwarden** at apply time — nothing secret
is committed:
```sh
bw login
export BW_SESSION=$(bw unlock --raw)   # then chezmoi apply
```
