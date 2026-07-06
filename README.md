# Tony's dev environment (chezmoi)

One declarative, cross-machine dev setup. Managed by [chezmoi](https://chezmoi.io):
`$HOME` files are generated from this repo, with per-OS templating, a single-source
colorblind-safe palette, and package installation.

## One-line install (fresh machine)

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply tonyferrell
```

Installs chezmoi, renders configs for the current OS, and installs the toolset —
non-interactive. macOS (Apple Silicon) + Ubuntu.

## Day-to-day

| Command | What it does |
|---|---|
| `chezmoi edit <file>` | edit the source of a managed file |
| `chezmoi apply` | render + write changes into `$HOME` |
| `chezmoi update` | `git pull` + apply |
| `chezmoi status` | show drift (files that differ from source) |
| `chezmoi cd` | drop into the source repo |

A **staleness check** on shell login warns when the checkout is behind its git remote.

## Layout

```
.chezmoidata/colors.yaml          Okabe-Ito CVD palette — single source of truth
dot_zshrc                         zsh: nix-first PATH, tool inits, aliases
dot_config/
  ghostty/config                  Ghostty (mac): CVD palette, OSC52, shift+enter fix
  tmux/tmux.conf.tmpl             tmux: pbcopy on mac / OSC52 on linux from one source
  nvim/                           kickstart.nvim base + CVD diagnostic colors
  git/config.tmpl                 delta + hunk + CVD diff colors
Brewfile                          macOS packages (brew bundle)
run_onchange_install-packages.sh.tmpl   installs the toolset (brew / cargo, no sudo)
```

## Colorblind (protanopia) design

All colors derive from `.chezmoidata/colors.yaml` (Okabe-Ito). Change it once and
Ghostty, tmux, git-delta, and nvim update together. "Red"→vermillion, "green"→teal,
so semantics are separable without the red/green axis.

## Code review workflow

`delta` is the git pager; `hunk` (`git review`) is the review-first TUI for agent
changesets; `lazygit` for hunk-staging; `tig`/`gh` for history/PRs.

## Per-machine customization

The base is common to every machine. Anything machine-specific stays **out of this repo**
and is pulled in only if the file exists locally:

- `~/.config/git/local.inc` — git user/email (`[include]`d by the base git config)
- `~/.zshrc.local` — shell env, aliases, extra-tool setup (sourced by the base `.zshrc`)
- `~/.Brewfile.local` — a few extra packages (`brew bundle`'d by the install hook)

Write these by hand per machine, or populate them from a separate private repo. Secrets
are never committed here — load them from your secret manager in `~/.zshrc.local`.
