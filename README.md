# scriptz

Scripts and general daily helpers.

## Installation

Most scripts here use [fzf](https://github.com/junegunn/fzf) for interactive
selection. Install it first:

```bash
# Debian or Ubuntu
sudo apt install fzf

# macOS
brew install fzf
```

Run `fzf --version` to check it works.

Clone the repo into `~/.local/share/scriptz`, the XDG data directory. The
symlinks point back into it, so moving or deleting the clone breaks them.

```bash
git clone git@github.com:kmjbyrne/scriptz.git ~/.local/share/scriptz
cd ~/.local/share/scriptz
```

Symlink each script you want into `~/.local/bin`. Use `-f` to replace an old
link:

```bash
mkdir -p ~/.local/bin
ln -sf ~/.local/share/scriptz/workflow/wt ~/.local/bin/wt
```

Check that `~/.local/bin` is on your `PATH`. If `command -v wt` prints nothing,
add this line to `~/.zshrc` or `~/.bashrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

As an alternative to symlinks, add a whole script folder to your `PATH`. Every
script in that folder becomes available, including new ones after a `git pull`.
You need one line per folder, because the shell does not search subfolders:

```bash
export PATH="$HOME/.local/share/scriptz/workflow:$PATH"
```

For `wt`, add this function to the same file, then open a new shell:

```bash
wt() { cd "$(command wt "$@")" || return; }
```

`command wt` skips the function and runs the script on your `PATH`, so the
function can share its name without calling itself.

To uninstall a script, remove its link with `rm ~/.local/bin/wt`.

## Scripts

Scripts are grouped by folder.

## Workflow

### Worktree Fuzzy GoGo

`wt`

Fuzzy-select a git worktree and `cd` into it. Run `wt` inside a repo, pick a
worktree from the fzf list, and your shell moves there. Cancel with `Esc` to
stay where you are.

Run `wt -` to jump back to the main worktree, the original clone that the other
worktrees hang off. It skips the picker.

Pass a search term to pre-fill the picker, as in `wt feature`. If only one
worktree matches, `wt` jumps straight there.

Add `--env` to bring your env files along. It copies every untracked `.env` and
`.env.*` file from the current worktree into the target, keeping their paths, so
`apps/api/.env.local` lands in the same place. Tracked files such as
`.env.example` and anything under `node_modules` are left out. Files that
already exist in the target are skipped, never overwritten. It works with both
forms:

```bash
wt --env
wt - --env
```

Pass a file name to copy only that file, wherever it sits in the tree. The name
can be a glob. `--env=*` is the same as plain `--env`. The old `--copy-env` flag
still works as an alias.

```bash
wt --env=.env.local
wt --env='.env.*.local'
```

The script itself only prints the selected path, because a script cannot change
its parent shell's directory. A small shell function of the same name does the
`cd`. See [Installation](#installation) for the setup.

It needs `git` and `fzf` on your `PATH`.

## License

MIT. See [LICENSE](LICENSE).
