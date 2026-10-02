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
ln -sf "$PWD/workflow/wtgo" ~/.local/bin/wtgo
ln -sf "$PWD/bin/pr-tree" ~/.local/bin/pr-tree
```

Check that `~/.local/bin` is on your `PATH`. If `command -v wtgo` prints
nothing, add this line to `~/.zshrc` or `~/.bashrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

As an alternative to symlinks, add a whole script folder to your `PATH`. Every
script in that folder becomes available, including new ones after a `git pull`.
You need one line per folder, because the shell does not search subfolders:

```bash
export PATH="$HOME/.local/share/scriptz/workflow:$PATH"
export PATH="$HOME/.local/share/scriptz/bin:$PATH"
```

For `wtgo`, add this function to the same file, then open a new shell:

```bash
wtgo() { cd "$(command wtgo "$@")" || return; }
```

`command wtgo` skips the function and runs the script on your `PATH`, so the
function can share its name without calling itself.

To uninstall a script, remove its link with `rm ~/.local/bin/wtgo`.

## Scripts

Scripts are grouped by folder.

## Workflow

### Worktree Fuzzy GoGo

`wtgo`

Fuzzy-select a git worktree and `cd` into it. Run `wtgo` inside a repo, pick a
worktree from the fzf list, and your shell moves there. Cancel with `Esc` to
stay where you are.

Run `wtgo -` to jump back to the main worktree, the original clone that the
other worktrees hang off. It skips the picker.

The script itself only prints the selected path, because a script cannot change
its parent shell's directory. A small shell function of the same name does the
`cd`. See [Installation](#installation) for the setup.

It needs `git` and `fzf` on your `PATH`.

## Bin

### PR Tree

`pr-tree`

Prints a file tree with commit summaries for a stack of PR branches. Useful for
visualizing what changed across a series of stacked PRs.

```bash
pr-tree <base-branch> <branch1> [branch2] [branch3] ...
```

Example with a 3-PR stack off main:

```bash
pr-tree main sign-fr-duplicate-client sign-fr-duplicate-domain sign-fr-duplicate-rest
```

Output groups changes per branch, shows commit subjects, and renders a collapsed
directory tree (single-child dirs are joined into one line, so deep package
paths stay compact). If the last branch is your current branch and has
uncommitted changes, those are included automatically.

It needs `git` and `python3` on your `PATH`.

## License

MIT. See [LICENSE](LICENSE).
