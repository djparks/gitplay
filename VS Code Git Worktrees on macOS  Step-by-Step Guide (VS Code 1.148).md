# VS Code Git Worktrees on macOS: A Step-by-Step Guide (VS Code 1.148)

This revision was checked against **VS Code 1.148.0** and **Git 2.54** on macOS. Commands, menus, and settings were confirmed against the Git extension that ships with VS Code 1.148. Menu names may differ in other versions.

A Git worktree is an extra working folder attached to the same repository. Each worktree has one branch checked out, with its own files, staging area, and uncommitted changes. All worktrees share the same history, branches, and remotes ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)).

## What changed from the earlier guide

| Topic | Earlier guide | VS Code 1.148 |
| --- | --- | --- |
| Where worktrees appear | Only as extra repositories in the Repositories view | Also in a **Worktrees** group inside each repository, when you turn on the experimental Repositories explorer (Step 1) |
| Right-click menu on a worktree | **Open Worktree in New Window / Current Window** | On a worktree *repository* row, the menu still shows those items plus **Delete Worktree**. On an item in the **Worktrees** group, it shows **Open**, **Open in New Window**, **Delete**, and **Copy Branch Name / Commit Hash / Worktree Path** |
| Default folder name | `<repo>.worktrees/<branch>` with `/` changed to `-` | Same, but the `git.branchPrefix` setting (if you set one) is removed first |
| **Git: Open Worktree in Current/New Window** in the Command Palette | Always available | Only appears when more than one repository is open in the window |
| Bare repo + worktrees layout | "Create and remove worktrees in Terminal" | Still true for the Command Palette commands. The **+** button in the **Worktrees** group also works from a linked worktree and defaults to a sibling folder (Part 2, Step 4) |
| **Migrate Worktree Changes** | "Moves changes into the primary worktree" | Same, but the changes are also **removed** from the source worktree. Run it from the main repository only |

---

## Worktrees vs. working sets

| | Working sets | Worktrees |
|---|---|---|
| Folders on disk | One folder; files change when you switch branches | One folder per branch |
| Uncommitted changes | Shared, so they can block a branch switch | Separate in each folder |
| Editors and layout per branch | Saved and restored on each switch | Automatic, because each folder is its own VS Code workspace |
| Run two branches at once | No | Yes (dev servers, tests, AI agents) |
| Disk use | Minimal | One full checkout per worktree (dependencies too, unless you share them) |

You don't need `scm.workingSets.enabled` (default `false`) with worktrees. Each folder already remembers its own tabs, splits, and cursor positions.

One rule to keep in mind: Git won't let the same local branch be checked out in two worktrees at the same time. VS Code shows an error and offers to open the worktree that already has the branch.

---

## Part 1: Standard worktrees (VS Code's built-in flow)

### Prerequisites

1. **VS Code.** These steps were checked on 1.148. Check your version in **Code > About Visual Studio Code**.
2. **Git.** Run `git --version`. Apple's Command Line Tools include Git, or you can run `brew install git`.
3. **The `code` command.** Press `Cmd+Shift+P` and run **Shell Command: Install 'code' command in PATH**.

### Step 1: Configure settings

Press `Cmd+Shift+P` and run **Preferences: Open User Settings (JSON)**. The file is at `~/Library/Application Support/Code/User/settings.json`.

```jsonc
{
  // Show worktrees that already exist (for example, ones created in Terminal).
  // Off by default.
  "git.detectWorktrees": true,
  "git.detectWorktreesLimit": 50,

  // Copy gitignored files that each worktree needs (Experimental)
  "git.worktreeIncludeFiles": [".env", ".env.local"],

  // Or symlink big gitignored folders instead of copying them (Experimental)
  "git.worktreeSymlinkFolders": ["node_modules/"],

  // Always show the Repositories section in Source Control
  "scm.alwaysShowRepositories": true,

  // NEW in recent versions (Experimental): show Branches, Tags, Stashes and
  // Worktrees under each repository. Works only with single selection mode.
  "scm.repositories.selectionMode": "single",
  "scm.repositories.explorer": true,

  // Put the folder name in the title bar so you can tell windows apart
  "window.title": "${rootName}${separator}${activeEditorShort}"
}
```

Notes:

- With `git.detectWorktrees` off, VS Code lists only the worktrees you create through its own UI.
- `scm.repositories.explorer` does nothing unless `scm.repositories.selectionMode` is `"single"`. With single selection, the Source Control view shows one repository at a time, and you click a repository in the Repositories section to switch.
- `git.worktreeIncludeFiles` copies only files that both match a pattern and are gitignored.
- `git.worktreeSymlinkFolders` links only folders that both match a pattern and are gitignored. A symlink shares one physical folder, so installing a package in one worktree changes it for every linked worktree. Use copying instead when branches need different dependencies.

### Step 2: Create a worktree in VS Code

Open the main repository folder, not a linked worktree, then use one of these methods.

**Method A: Repository "…" menu**

1. Open the **Source Control** view (`Ctrl+Shift+G`).
2. In **Repositories**, hover over the main repository and click **More Actions (…)**.
3. Choose **Worktrees > Create Worktree…**.

**Method B: Worktrees group (needs the explorer settings from Step 1)**

1. In **Repositories**, expand the main repository, then hover over **Worktrees**.
2. Click the **+** (**Create Worktree…**) button.

**Method C: Command Palette.** Run **Git: Create Worktree…**.

Then:

4. Pick a branch or tag to start from, or choose to create a new branch.
5. Accept or edit the folder path. The folder button next to the path box opens a folder picker.

The default path is a sibling folder named `<repo>.worktrees/<branch-name>`. Slashes in the branch name become dashes, so `feature/login` becomes `feature-login`. If you set `git.branchPrefix` (for example `"djp/"`), the prefix is left out of the folder name. If a worktree already exists at that path, VS Code adds `-1`, `-2`, and so on. After you choose a different parent folder once, VS Code uses it as the default for that repository from then on.

```
~/code/
├── myapp/                    ← primary worktree (main)
└── myapp.worktrees/
    ├── feature-login/        ← linked worktree
    └── hotfix-crash/
```

If you set `git.worktreeIncludeFiles` or `git.worktreeSymlinkFolders`, VS Code copies or links those files right after it creates the folder.

### Step 3: Open and switch between worktrees

- **From a worktree repository row:** In **Repositories**, right-click a worktree (it has a worktree icon), then choose **Open Worktree in New Window** or **Open Worktree in Current Window**. These items are also under **More Actions (…) > Worktrees**.
- **From the Worktrees group (explorer on):** Hover over a worktree and click the inline **Open in New Window** icon, or right-click it and choose **Open** or **Open in New Window**. The right-click menu can also copy the branch name, commit hash, or path.
- **From the Command Palette:** **Git: Open Worktree in New Window** and **Git: Open Worktree in Current Window** appear only when more than one repository is open in the window.
- **From Terminal:** Run `code ~/code/myapp.worktrees/feature-login`.
- **To jump back:** Use **File > Open Recent** (`Ctrl+R` on macOS).

Each window keeps its own tabs, splits, terminals, and cursor positions.

### Step 4: Compare and move changes

- **Compare a file with the main checkout.** In the primary repository's window, find the worktree's repository in **Repositories** (this needs `git.detectWorktrees`). Under that worktree's **Changes**, right-click a file and choose **Compare with Workspace**. VS Code compares the worktree's file with the same file in the main repository folder.
- **Move uncommitted work.** In the primary repository, run **Git: Migrate Worktree Changes…** and pick the worktree. VS Code asks you to confirm, then moves the staged, unstaged, and **untracked** changes into the main folder and **removes them from the worktree**. If the changes conflict, VS Code can open them in the merge editor. This command doesn't move commits. To bring commits over, merge the branch.

### Step 5: Remove a worktree

1. Commit, stash, or copy out anything you want to keep.
2. Work from the primary repository's window, not the worktree you're removing.
3. Use one of these:
   - **Command Palette:** **Git: Delete Worktree…**, then pick the worktree.
   - **Worktree repository row:** Right-click it and choose **Delete Worktree**.
   - **Worktrees group (explorer on):** Right-click it and choose **Delete**.
4. Check the path before you confirm.

If you try to delete the worktree that's open in the current window, VS Code shows "You cannot delete the worktree you are currently in. Please switch to the main repository first."

Deleting a worktree removes its folder, including gitignored files like `.env`. The branch and its commits stay in the repository. If the worktree has modified or untracked files, VS Code asks whether to **Force Delete**, which discards them.

Terminal equivalent:

```bash
git worktree list
git worktree remove ../myapp.worktrees/feature-login
git worktree prune          # clears records for folders you deleted by hand
```

---

## Part 2: Repository in the root folder, branches in subfolders

This is often called the **bare repo + worktrees** layout ([joncardasis gist](https://gist.github.com/joncardasis/a08f2d5d4f2e576dc703b0638ef81218), [Tom Ups](https://www.tomups.com/posts/git-worktrees/)). The root folder holds only the Git database. Every branch, including `main`, is a sibling subfolder:

```
~/code/myapp/                  ← container (not a checkout)
├── .bare/                     ← bare repository: objects, refs, config
├── .git                       ← one-line FILE: gitdir: ./.bare
├── myapp.code-workspace       ← optional multi-root workspace
├── main/                      ← worktree for main
├── feature-login/             ← worktree for feature/login
└── hotfix-crash/              ← worktree for hotfix/crash
```

Because the `.git` file points at `.bare`, Git commands work from the root folder, so you can run `git worktree add feature-login` without typing `../` paths.

### VS Code limits in 1.148

- **Open a branch folder, not the root.** VS Code doesn't treat the container folder as a working repository. Open `main/`, `feature-login/`, and so on.
- **Every branch folder is a linked worktree.** VS Code handles this better than before. It lists the other worktrees, skips the bare database, and labels each worktree with the repository's name. Bare-repository support is still incomplete ([#267606](https://github.com/microsoft/vscode/issues/267606)).
- **Command Palette commands don't work here.** **Git: Create Worktree…**, **Git: Delete Worktree…**, and **Git: Migrate Worktree Changes…** run only on a main repository or submodule, so in this layout they won't find a repository to act on.
- **The Worktrees group does work.** With the explorer settings from Part 1, Step 1, the **+** button in the **Worktrees** group can create a worktree from a linked worktree. **Delete** there can remove a worktree other than the one you have open. See Step 4.
- **Use absolute worktree paths.** Worktrees created with Git's `--relative-paths` option have had detection bugs in VS Code ([#257396](https://github.com/microsoft/vscode/issues/257396)). Git uses absolute paths by default.

Editing, committing, diffing, pushing, and pulling work normally in each branch folder.

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

Step 3 is needed because `git clone --bare` doesn't set up remote-tracking branches.

### Step 2: Convert an existing repository

Commit or stash everything first. The safest way is to re-clone into the new layout, using your existing local folder as the source:

```bash
cd ~/code
git clone --bare ./myapp myapp-wt/.bare
cd myapp-wt
printf 'gitdir: ./.bare\n' > .git
git config remote.origin.url "$(git -C ../myapp remote get-url origin)"
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch origin
git worktree add main main
git -C main branch --set-upstream-to=origin/main main
```

Make sure everything is there, then archive the old folder.

### Step 3: Add branch folders in Terminal

Run these from the root folder:

```bash
# Existing remote branch
git worktree add feature-login feature/login

# Brand-new branch based on main
git worktree add -b feature/search feature-search origin/main

# See them all
git worktree list
```

You can add this helper to `~/.zshrc`. It creates a worktree, copies `.env`, and opens it in VS Code. It's meant for the bare layout only. In a normal repository it would create the folder inside your checkout.

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

### Step 4: Add branch folders from VS Code

1. Turn on `scm.repositories.selectionMode: "single"` and `scm.repositories.explorer: true` (Part 1, Step 1).
2. Open `~/code/myapp/main` in VS Code.
3. In **Source Control > Repositories**, expand the repository, hover over **Worktrees**, and click **+**.
4. Pick a branch or create a new one. Because you're in a linked worktree, the default path is a **sibling folder**, for example `~/code/myapp/feature-login`. That's the layout you want.

This uses an experimental view. If it doesn't work in your setup, use Step 3.

### Step 5: Open in VS Code

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

Open it with `code ~/code/myapp/myapp.code-workspace`. Each folder appears as its own repository in Source Control, and you choose which one to commit to. Because several repositories are open, **Git: Open Worktree in New Window** and **Git: Open Worktree in Current Window** also appear in the Command Palette.

If repository names show up wrong or twice, make sure the workspace file uses relative folder paths ([#257396](https://github.com/microsoft/vscode/issues/257396)).

### Step 6: Keep it clean

```bash
git worktree remove feature-login     # deletes the folder; the branch stays
git branch -d feature/login           # delete the branch after merging
git worktree prune                    # clean up after manual folder deletes
git fetch --prune origin              # sync remote-tracking branches
```

When you delete a worktree, also remove its entry from the `.code-workspace` file if you use one.

---

## Part 3: Color-code worktree windows

Each worktree opens in its own window, so you can give each window its own title bar, status bar, and activity bar colors. A common setup is to leave `main` with your normal theme and color the others, for example green and yellow:

```text
gitplay/                       ← main: normal theme (no color file)
gitplay.worktrees/
├── Branch2/.vscode/settings.json        ← green
└── hotfix-crash/.vscode/settings.json   ← yellow
```

Colors apply to a whole window. In a multi-root workspace (Part 2, Step 5, Option B), every folder in the window shares the same colors, so use one window per worktree if you want them color-coded.

### Step 1: Keep the color file out of Git

The colors live in each worktree's `.vscode/settings.json`. If that file were committed, the color would follow the branch, and merging a green branch would turn `main` green. Exclude it once for the whole repository:

```bash
# Run from any worktree of the repository
printf '\n# Per-worktree VS Code settings (window colors)\n.vscode/settings.json\n' \
  >> "$(git rev-parse --path-format=absolute --git-common-dir)/info/exclude"
```

`info/exclude` works like `.gitignore`, but it's never committed and only exists on your computer. All worktrees of a repository share it, so you only do this once. Check it with:

```bash
git check-ignore -v .vscode/settings.json
```

**If your repository already tracks `.vscode/settings.json`** (for example, for shared formatter settings), excluding it has no effect. Add the color block to that worktree's copy instead, then run this inside the worktree:

```bash
git update-index --skip-worktree .vscode/settings.json
```

This tells Git to ignore your local changes to that file in this worktree only. To undo it, run `git update-index --no-skip-worktree .vscode/settings.json`.

### Step 2: Add colors to a worktree

Create `.vscode/settings.json` in the worktree folder. Don't create one in the main folder.

Green (white text):

```jsonc
{
  "workbench.colorCustomizations": {
    "titleBar.activeBackground": "#2e7d32",
    "titleBar.activeForeground": "#ffffff",
    "titleBar.inactiveBackground": "#2e7d32cc",
    "titleBar.inactiveForeground": "#ffffffcc",
    "statusBar.background": "#2e7d32",
    "statusBar.foreground": "#ffffff",
    "activityBar.background": "#1b5e20"
  }
}
```

Yellow (black text, so it stays readable):

```jsonc
{
  "workbench.colorCustomizations": {
    "titleBar.activeBackground": "#f9a825",
    "titleBar.activeForeground": "#000000",
    "titleBar.inactiveBackground": "#f9a825cc",
    "titleBar.inactiveForeground": "#000000cc",
    "statusBar.background": "#f9a825",
    "statusBar.foreground": "#000000",
    "activityBar.background": "#f57f17"
  }
}
```

The window changes color as soon as you save; you don't need to reload. The `cc` at the end of the inactive colors makes them slightly transparent, so a window you aren't using looks a little faded. If you only want a hint of color, keep just the `statusBar.*` lines.

The title bar colors work with VS Code's default title bar style on macOS (`"window.titleBarStyle": "custom"`). If you changed it to `"native"`, the title bar uses the macOS colors and only the status bar and activity bar change.

### Step 3 (optional): A `wtcolor` helper

Add this to `~/.zshrc`. Run it inside any worktree to write that worktree's color file:

```zsh
# Usage: wtcolor 2e7d32          (green, white text)
#        wtcolor f9a825 000000   (yellow, black text)
wtcolor() {
  local bg="${1#\#}" fg="${2:-ffffff}"; fg="${fg#\#}"
  if [[ ! $bg =~ ^[0-9a-fA-F]{6}$ || ! $fg =~ ^[0-9a-fA-F]{6}$ ]]; then
    echo "Usage: wtcolor <bg-hex> [fg-hex]   e.g. wtcolor 2e7d32  or  wtcolor f9a825 000000"; return 1
  fi
  local top; top="$(git rev-parse --show-toplevel)" || return 1
  local file="$top/.vscode/settings.json"
  if [[ -e $file ]] && ! grep -q '"workbench.colorCustomizations"' "$file"; then
    echo "$file already has other settings. Add the colors by hand."; return 1
  fi
  mkdir -p "$top/.vscode"
  cat > "$file" <<JSON
{
  // Worktree color. Excluded from Git via .git/info/exclude.
  "workbench.colorCustomizations": {
    "titleBar.activeBackground": "#$bg",
    "titleBar.activeForeground": "#$fg",
    "titleBar.inactiveBackground": "#${bg}cc",
    "titleBar.inactiveForeground": "#${fg}cc",
    "statusBar.background": "#$bg",
    "statusBar.foreground": "#$fg",
    "activityBar.background": "#$bg"
  }
}
JSON
  git -C "$top" check-ignore -q .vscode/settings.json \
    || echo "Warning: .vscode/settings.json is not ignored. Add it to $(git rev-parse --path-format=absolute --git-common-dir)/info/exclude."
  echo "Wrote $file"
}
```

The helper:

- Refuses to overwrite a `.vscode/settings.json` that holds other settings.
- Warns you if the file isn't excluded from Git (Step 1).

It works with both layouts. In Part 2, you can run it right after the `wt` helper:

```bash
wt hotfix/crash && (cd ~/code/myapp/hotfix-crash && wtcolor f9a825 000000)
```

**New worktrees don't inherit colors.** Gitignored files aren't copied to new worktrees, and VS Code's **Create Worktree** only copies files listed in `git.worktreeIncludeFiles`. Leave `.vscode/settings.json` out of that setting. If you add it, a worktree created from a colored worktree gets that worktree's color too. Run `wtcolor` in each new worktree instead.

**Alternative:** The Peacock extension sets these same colors from a color picker. It writes them to the same `.vscode/settings.json` file, so Step 1 still applies.

---

## Troubleshooting

- **A worktree doesn't show in Source Control.** Turn on `git.detectWorktrees`, then reload the window (`Cmd+Shift+P`, then **Developer: Reload Window**). Or open the worktree folder directly.
- **A worktree window has no color, or the wrong one.** Make sure you opened the worktree folder itself, not a multi-root workspace. Then make sure the worktree's `.vscode/settings.json` contains `workbench.colorCustomizations` (Part 3).
- **`.vscode/settings.json` shows up as a change to commit.** It isn't excluded. Follow Part 3, Step 1.
- **There's no Worktrees group under the repository.** Set both `scm.repositories.selectionMode` to `"single"` and `scm.repositories.explorer` to `true`.
- **Git: Create Worktree… does nothing or says there's no repository.** The open folder is a linked worktree. Open the main repository, or use the **+** in the **Worktrees** group (Part 2, Step 4).
- **Git: Open Worktree… isn't in the Command Palette.** It appears only when more than one repository is open. Use the Repositories view instead.
- **"Branch is already checked out."** That branch is open in another worktree. Run `git worktree list` to find it, or accept VS Code's offer to open that worktree.
- **Missing `.env` or `node_modules`.** Gitignored files aren't copied to new worktrees. Use `git.worktreeIncludeFiles`, `git.worktreeSymlinkFolders`, or the `wt` helper's copy step.
- **High CPU or disk use.** VS Code, language servers, and file watchers index each worktree separately. Close windows you aren't using, and prune old worktrees.
- **"invalid gitfile format."** The `.git` file must contain exactly `gitdir: ./.bare`. Create it with `printf`, as shown above.
- **You moved a worktree folder in Finder.** Run `git worktree repair` from the new location, because Git stores absolute paths ([git-worktree docs](https://git-scm.com/docs/git-worktree.html)).
