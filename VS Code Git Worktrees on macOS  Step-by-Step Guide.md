# VS Code Git Worktrees on macOS: A Step-by-Step Guide

A Git worktree is an extra working folder attached to the same repository. Each worktree has one branch checked out, with its own files, staging area, and uncommitted changes. All worktrees share the same history, branches, and remotes ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)).

## Worktrees vs. working sets

| | Working sets | Worktrees |
|---|---|---|
| Folders on disk | One folder; files change when you switch branches | One folder per branch |
| Uncommitted changes | Shared, so they can block a branch switch | Separate in each folder |
| Editors and layout per branch | Saved and restored on each switch | Automatic, because each folder is its own VS Code workspace |
| Run two branches at once | No | Yes (dev servers, tests, AI agents) |
| Disk use | Minimal | One full checkout per worktree (dependencies too, unless you share them) |

With worktrees, you don't need `scm.workingSets.enabled`. Each folder already remembers its own tabs, splits, and cursor positions.

One rule to keep in mind: Git won't let the same local branch be checked out in two worktrees at the same time ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)).

---

## Part 1: Standard worktrees (VS Code's built-in flow)

### Prerequisites

1. **VS Code.** Use a recent release. Worktree support first shipped in [VS Code 1.103](https://github.com/Microsoft/vscode/issues/68038) and has improved since.
2. **Git.** Run `git --version` and make sure you have 2.5 or later. Apple's Command Line Tools work, or run `brew install git` for a newer version.
3. **The `code` command.** Press `Cmd+Shift+P` and run **Shell Command: Install 'code' command in PATH**.

### Step 1: Configure settings

Press `Cmd+Shift+P` and run **Preferences: Open User Settings (JSON)**. The file is at `~/Library/Application Support/Code/User/settings.json`.

```jsonc
{
  // Show worktrees that already exist (for example, ones created in Terminal)
  "git.detectWorktrees": true,
  "git.detectWorktreesLimit": 50,

  // Copy gitignored files that each worktree needs (Experimental)
  "git.worktreeIncludeFiles": [".env", ".env.local"],

  // Or symlink big gitignored folders instead of copying them (Experimental)
  "git.worktreeSymlinkFolders": ["node_modules/"],

  // Always show the Repositories view so worktrees are easy to see
  "scm.alwaysShowRepositories": true,

  // Put the folder name in the title bar so you can tell windows apart
  "window.title": "${rootName}${separator}${activeEditorShort}"
}
```

`git.detectWorktrees` is off by default. Without it, VS Code only lists the worktrees you create through its own UI ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)).

A note on the two Experimental settings: `git.worktreeIncludeFiles` copies only files that are both matched by a pattern and gitignored. `git.worktreeSymlinkFolders` shares one physical folder, so installing a package in one worktree affects every worktree linked to it ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)). Use copy, not symlink, when branches need different dependencies.

### Step 2: Create a worktree in VS Code

1. Open the main repository folder.
2. Open the **Source Control** view (`Ctrl+Shift+G`), then expand **Repositories**.
3. On the repository, open **More Actions (…)**, then choose **Worktrees > Create Worktree…**.
4. Pick an existing branch, or create a new one.
5. Accept or edit the path.

The default path is a sibling folder named `<repo>.worktrees/<branch-name>`. Slashes in the branch name become dashes, so `feature/login` becomes `feature-login`. After you pick a different location once, VS Code remembers it for that repository ([`commands.ts`](https://raw.githubusercontent.com/microsoft/vscode/main/extensions/git/src/commands.ts)).

```
~/code/
├── myapp/                    ← primary worktree (main)
└── myapp.worktrees/
    ├── feature-login/        ← linked worktree
    └── hotfix-crash/
```

### Step 3: Open and switch between worktrees

- **From the Repositories view:** Right-click a worktree and choose **Open Worktree in New Window** or **Open Worktree in Current Window**.
- **From Terminal:** Run `code ~/code/myapp.worktrees/feature-login`.
- **To jump back:** Use **File > Open Recent** (`Ctrl+R`).

Each window keeps its own tabs, splits, terminals, and cursor positions.

### Step 4: Compare and move changes

- **Compare a file.** In the primary repository's window, select the worktree in the Repositories view. Then right-click a changed file and choose **Compare with Workspace**.
- **Move uncommitted work.** Run **Git: Migrate Worktree Changes…** to move uncommitted changes, including untracked files, into the primary worktree. This doesn't merge commits. To bring commits over, merge the branch ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)).

### Step 5: Remove a worktree

1. Commit, stash, or copy out anything you want to keep.
2. Open the primary repository's folder, not the worktree you're removing.
3. Run **Git: Delete Worktree…** and check the displayed path before you confirm.

Deleting a worktree removes its folder, including gitignored files like `.env`. The branch and its commits stay in the repository. **Force Delete** discards uncommitted changes ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)).

Terminal equivalent:

```bash
git worktree list
git worktree remove ../myapp.worktrees/feature-login
git worktree prune          # clears records for folders you deleted by hand
```

---

## Part 2: Repository in the root folder, branches in subfolders

Yes, this is possible. The common name is the **bare repo + worktrees** layout ([joncardasis gist](https://gist.github.com/joncardasis/a08f2d5d4f2e576dc703b0638ef81218), [Tom Ups](https://www.tomups.com/posts/git-worktrees/)). The root folder holds only the Git database. Every branch, including `main`, is a sibling subfolder:

```
~/code/myapp/                  ← container (not a checkout)
├── .bare/                     ← bare repository: objects, refs, config
├── .git                       ← one-line FILE: gitdir: ./.bare
├── myapp.code-workspace       ← optional multi-root workspace
├── main/                      ← worktree for main
├── feature-login/             ← worktree for feature/login
└── hotfix-crash/              ← worktree for hotfix/crash
```

Because the `.git` file points at `.bare`, Git commands work from the root folder. That means you can run `git worktree add feature-login` without typing `../` paths ([Tom Ups](https://www.tomups.com/posts/git-worktrees/)).

### Know the VS Code limits first

- **Bare repositories aren't officially supported.** The VS Code issue ["Git – better support for bare repositories" (#267606)](https://github.com/microsoft/vscode/issues/267606) is still open and in the Backlog. Users report inconsistent worktree detection with bare repositories.
- **Open a branch folder, not the root.** If you open the container folder as the root, VS Code's Git integration may not detect the worktrees ([Cline #7052](https://github.com/cline/cline/issues/7052)).
- **Expect some UI commands to be missing.** VS Code's **Create Worktree** command only runs against repositories of type `repository` or `submodule`, not `worktree` ([`commands.ts`](https://raw.githubusercontent.com/microsoft/vscode/main/extensions/git/src/commands.ts)). In this layout every folder is a linked worktree, so create and remove worktrees in Terminal.
- **Avoid relative worktree paths for now.** Worktrees created with Git's `--relative-paths` option have had detection bugs in VS Code ([#257396](https://github.com/microsoft/vscode/issues/257396), [#267606](https://github.com/microsoft/vscode/issues/267606)). Keep the default absolute paths.

Day-to-day editing, committing, diffing, pushing, and pulling all work normally inside each branch folder.

### Step 1: Set up from a remote

```bash
mkdir -p ~/code/myapp && cd ~/code/myapp

# 1. Clone the Git database only (no working files)
git clone --bare git@github.com:you/myapp.git .bare

# 2. Point Git at it from the root folder
printf 'gitdir: ./.bare\n' > .git

# 3. Bare clones don't track remote branches by default, so fix fetch
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch origin

# 4. Create the main worktree and set upstream tracking
git worktree add main main
git -C main branch --set-upstream-to=origin/main main
```

Step 3 is necessary because `git clone --bare` doesn't configure remote-tracking branches ([Homelab Adventures](https://hai.wxs.ro/guides/git-worktrees/)).

### Step 2: Convert an existing repository

Commit or stash everything first. The simplest safe path is to re-clone into the new layout, using your existing local folder as the source:

```bash
cd ~/code
git clone --bare ./myapp myapp-wt/.bare
cd myapp-wt
printf 'gitdir: ./.bare\n' > .git
git config remote.origin.url "$(git -C ../myapp remote get-url origin)"
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch origin
git worktree add main main
```

Check that everything is there, then archive the old folder.

### Step 3: Add branch folders

Run these from the root folder:

```bash
# Existing remote branch
git worktree add feature-login feature/login

# Brand-new branch based on main
git worktree add -b feature/search feature-search origin/main

# See them all
git worktree list
```

You can add this helper to `~/.zshrc`. It creates a worktree, copies `.env`, and opens it in VS Code:

```bash
wt() {
  local branch="$1" base="${2:-origin/main}"
  local root; root="$(git rev-parse --path-format=absolute --git-common-dir)/.."
  local dir="$root/${branch//\//-}"
  if git show-ref --verify --quiet "refs/heads/$branch"; then
    git worktree add "$dir" "$branch"
  else
    git worktree add -b "$branch" "$dir" "$base"
  fi
  [ -f "$root/main/.env" ] && cp "$root/main/.env" "$dir/"
  code "$dir"
}
```

Usage: `wt feature/login`, or `wt hotfix/crash origin/release-2.3`.

### Step 4: Open in VS Code

**Option A: One window per branch (most reliable)**

```bash
code ~/code/myapp/main
code ~/code/myapp/feature-login
```

**Option B: One multi-root window for the whole project**

Save this as `~/code/myapp/myapp.code-workspace`:

```jsonc
{
  "folders": [
    { "name": "main", "path": "main" },
    { "name": "feature-login", "path": "feature-login" }
  ],
  "settings": {
    "git.detectWorktrees": true,
    "scm.alwaysShowRepositories": true,
    "files.exclude": { "**/.bare": true }
  }
}
```

Open it with `code ~/code/myapp/myapp.code-workspace`. Each folder appears as its own repository in Source Control, and you pick which one to commit to.

If repository names show up wrong or duplicated, check that the workspace file uses relative folder paths. A user reported that this fixed the problem ([#257396](https://github.com/microsoft/vscode/issues/257396)).

### Step 5: Keep it clean

```bash
git worktree remove feature-login     # deletes the folder; the branch stays
git branch -d feature/login           # delete the branch after merging
git worktree prune                    # clean up after manual folder deletes
git fetch --prune origin              # sync remote-tracking branches
```

If you add worktrees to a multi-root workspace, remove the matching entry from the `.code-workspace` file when you delete one.

---

## Troubleshooting

- **A worktree doesn't show in Source Control.** Turn on `git.detectWorktrees`, then reload the window (`Cmd+Shift+P`, then **Developer: Reload Window**). Or open the worktree folder directly.
- **"Branch is already checked out."** That branch is open in another worktree. Run `git worktree list` to find it.
- **Missing `.env` or `node_modules`.** Gitignored files aren't copied to new worktrees. Use `git.worktreeIncludeFiles` or `git.worktreeSymlinkFolders`, or the `wt` helper's copy step.
- **High CPU or disk use.** Each worktree is indexed separately by VS Code, language servers, and file watchers ([recca0120](https://recca0120.github.io/en/2026/04/14/git-worktree-parallel-work/)). Close windows you aren't using, and prune old worktrees.
- **"invalid gitfile format."** The `.git` file must contain exactly `gitdir: ./.bare`. Create it with `printf`, as shown above.
- **You moved a worktree folder in Finder.** Run `git worktree repair` from the new location, because Git stores absolute paths ([git-worktree docs](https://git-scm.com/docs/git-worktree.html)).
- **GitLens.** If you use GitLens, its Worktrees view works with the `.bare` plus `.git` file layout ([GitLens #3090](https://github.com/gitkraken/vscode-gitlens/issues/3090)).
