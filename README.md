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
ln -sf "$PWD/workflow/wt" ~/.local/bin/wt
ln -sf "$PWD/workflow/wtgc" ~/.local/bin/wtgc
ln -sf "$PWD/bin/pr-tree" ~/.local/bin/pr-tree
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
export PATH="$HOME/.local/share/scriptz/bin:$PATH"
```

For `wt` and `wtnew`, add these functions to the same file, then open a new
shell:

```bash
wt() { cd "$(command wt "$@")" || return; }
wtnew() { cd "$(command wtnew "$@")" || return; }
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

### Worktree New

`wtnew`

Create a worktree and `cd` into it. Run `wtnew feat/search` to name the branch,
or plain `wtnew` to fuzzy-pick one. The picker lists local branches without a
worktree and `origin` branches you don't have locally. Type a name that matches
nothing to create a new branch. A new branch starts from the main worktree's
`HEAD`, and an `origin` branch is checked out with tracking set up.

The worktree goes in `<main>/.claude/worktrees/<branch>`, where `claude -w` puts
its own, with `/` in the branch name turned into `-`. Untracked `.env` and
`.env.*` files from the main worktree are copied in.

Add `--claude` to start Claude in the new worktree, with an optional first
prompt. When Claude exits, your shell is in the worktree:

```bash
wtnew feat/search --claude "Add a search box to the header"
```

Like `wt`, it needs the shell function from [Installation](#installation) to
`cd`. It needs `git` and `fzf` on your `PATH`, plus `claude` for `--claude`.

### Worktree Claude

`wtc`

Fuzzy-pick a worktree and resume its latest Claude conversation there. A `●`
marks worktrees that have one. Those without one start a fresh conversation. The
preview shows recent commits and the newest note in `instance/next`.

```bash
# Pick, then resume
wtc

# Pre-fill the picker, and skip it on a single match
wtc search

# Start fresh even if a conversation exists
wtc -n

# Pass flags through to claude
wtc -- --model opus
```

Claude Code stores conversations under `~/.claude/projects`, in a folder named
after the directory. `wtc` relies on that naming to find them, so a change in
Claude Code could break the `●` marks.

It needs `git`, `fzf` and `claude` on your `PATH`.

### Worktree Next

`wtnext`

Treat handover notes as a queue. It lists every `instance/next/<date>/*.md` note
across all worktrees, newest first, with the note in the preview. Pick one, and
Claude starts in that note's worktree with the note as its task. Pass a search
term to pre-fill the picker, and flags for `claude` after `--`.

It doesn't track which notes are done. Delete or move a note once it's handled,
or it stays in the list.

It needs `git`, `fzf` and `claude` on your `PATH`.

### Worktree Fuzzy Garbage Collect

`wtgc`

Fuzzy-select local branches and detached worktrees, and delete each one
everywhere: its worktree on disk, the local branch, and the branch on `origin`.
Press `Tab` to pick more than one. The script lists what it will delete and asks
before it touches anything.

```bash
# Safe: skips dirty worktrees and unmerged branches
wtgc

# List only merged branches and worktrees
wtgc --merged

# Force past both
wtgc -f

# Skip the confirmation
wtgc -y

# Rescue folders before deleting
wtgc --merge instance --merge notes

# Rescue instance, but leave instance/var behind
wtgc --merge instance --exclude instance/var
```

`--merge PATH` copies `PATH` out of each worktree into the same path under the
current worktree before the worktree is deleted. `wtgc --merge instance` moves
`<worktree>/instance` into `./instance`, merging with what is already there.
Pass it more than once for several paths. When a file exists on both sides with
different content, yours stays and the rescued copy lands beside it as
`<name>.<branch>`, so nothing is lost.

Add `-v` to list each file salvaged. Clashing files are always listed.

`--exclude PATH` leaves a path out of the rescue, and can be a glob.
`--merge '!PATH'` does the same, but needs single quotes. zsh expands `!` even
inside double quotes.

Each entry shows how it's merged. `✓ git` means its commits are in the main
worktree's `HEAD` or `origin`'s default branch. `✓ pr` means GitHub reports a
merged PR for it, which catches squash merges. A branch only counts as `✓ pr`
when its tip is part of that PR, so commits added after the merge keep it
unmerged. Being pushed to its own upstream doesn't count, because `wtgc` deletes
that too. Without `gh`, only `✓ git` is checked.

An unmerged entry is skipped before anything is touched, so its worktree stays
put. Use `-f` to delete it anyway. Detached worktrees, such as ones left by
background Claude jobs, are listed too.

The list leaves out the main worktree's branch, the branch you are on, and the
`origin` default branch. If a worktree or local branch can't be removed, that
branch is skipped and nothing is pushed for it. The script exits non-zero if
anything was skipped.

It needs `git` and `fzf` on your `PATH`, plus `gh` for `✓ pr`.

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
