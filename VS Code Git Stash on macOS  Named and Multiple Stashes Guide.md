# VS Code Git Stash on macOS: Named and Multiple Stashes

This guide was checked against **VS Code 1.148.0** and **Git 2.54** on macOS. Commands, menus, and settings were confirmed against the Git extension that ships with VS Code 1.148. The Git behaviors described below were tested in a scratch repository.

A stash saves your uncommitted changes and resets your files to the last commit, so you can switch tasks without making a throwaway commit. Later you can bring the changes back.

---

## How multiple stashes work

Git keeps stashes in a **stack**. The newest stash is always `stash@{0}`:

```text
stash@{0}: On main: gamma          ← newest
stash@{1}: On feature/login: beta
stash@{2}: On main: alpha          ← oldest
```

Three things to know:

1. **Numbers shift.** When you create a stash, every existing stash moves down one number. When you drop or pop `stash@{0}`, every other stash moves up one. `stash@{1}` today may be `stash@{2}` tomorrow, so **identify stashes by their message, not their number**.
2. **Messages are the names.** A stash with a message appears as `On <branch>: <message>`. Without a message, Git makes one up, for example `WIP on main: 3f2a1c9 Fix login bug`, so it's hard to tell several of those apart. Always add a message.
3. **Stashes belong to the repository, not the branch.** You can apply a stash made on `feature/login` while you're on `main`. Every worktree of the repository also sees the same list.

---

## Step 1: Configure settings

Press `Cmd+Shift+P` and run **Preferences: Open User Settings (JSON)**.

```jsonc
{
  // Ask about unsaved editors before stashing. Options:
  // "always" (default), "staged" (only staged files), "never".
  // Unsaved editor changes are NOT included in a stash.
  "git.promptToSaveFilesBeforeStash": "always",

  // Pre-fill the stash message with the text in the commit message box.
  // Off by default.
  "git.useCommitInputAsStashMessage": true,

  // Stash before Pull and restore afterward (git pull --autostash). Off by default.
  "git.autoStash": false,

  // Experimental: show a Stashes list (plus Branches, Tags, Worktrees)
  // under each repository in Source Control. Both settings are required.
  "scm.alwaysShowRepositories": true,
  "scm.repositories.selectionMode": "single",
  "scm.repositories.explorer": true
}
```

The **Stashes** list is the easiest way to manage several stashes. The rest of this guide covers it as well as the menus.

---

## Step 2: Create a named stash

1. Open the **Source Control** view (`Ctrl+Shift+G`).
2. Click **More Actions (…)** at the top of the view, or on the repository row, then open **Stash**.
3. Choose one of these:

| Menu item | What it stashes | Git command |
| --- | --- | --- |
| **Stash** | Staged and unstaged changes to tracked files | `git stash push -m "<msg>"` |
| **Stash (Include Untracked)** | The same, plus new files Git doesn't track yet | `git stash push -u -m "<msg>"` |
| **Stash Staged** | Only the staged changes | `git stash push -S -m "<msg>"` |

After you choose, a box appears that says **Optionally provide a stash message**. Type a name, for example `login: half-done validation`, and press `Return`.

- Press `Return` with the box empty to stash without a name. You get a `WIP on …` entry.
- Press `Escape` to cancel. Nothing is stashed.

You can also use these from the Command Palette: **Git: Stash**, **Git: Stash (Include Untracked)**, and **Git: Stash Staged**.

With the Stashes list turned on, hover over **Stashes** in the repository's tree and click its **+** menu. It offers **Stash**, **Stash Staged**, and **Stash (Include Untracked)**.

**Things VS Code checks or does for you:**

- If any editors for files in the repository have unsaved changes, VS Code warns that those changes **won't be included in the stash**. It offers **Save All & Stash** or **Stash Anyway**.
- If there's nothing to stash, you see "There are no changes to stash."
- A new repository needs at least one commit before you can stash.
- After stashing, VS Code closes the diff editors for the stashed files.

### Plain "Stash" doesn't include new files

Plain **Stash** leaves untracked files in your folder. If you're stashing to switch to a different task, you usually want **Stash (Include Untracked)**. Ignored files, such as `.env` or `node_modules`, are never stashed.

### Stashing only some files

VS Code has no "stash these files" command. You have two options:

- **Stage, then Stash Staged.** Stage only the files (or lines) you want to stash, then run **Stash Staged**. Your other changes stay in place.
- **Terminal:**

  ```bash
  git stash push -m "only the API client" -- src/api/client.ts src/api/types.ts
  ```

---

## Step 3: See all your stashes

#### Option A: The Stashes list (needs the experimental settings from Step 1)

In **Source Control > Repositories**, expand the repository, then expand **Stashes**. Each item shows its message. Right-click a stash to get:

- **View Stash**
- **Apply Stash** or **Pop Stash**
- **Drop Stash**
- **Copy Branch Name** or **Copy Stash Name**

Hovering over a stash shows an inline **Apply Stash** button. Hold `Option` to change it to **Pop Stash**.

#### Option B: View Stash…

1. Run **Git: View Stash…** from the Command Palette, or use **More Actions (…) > Stash > View Stash…**.
2. The picker lists each stash as `#<number>: <message>`. Beside each one it shows how long ago the stash was made and which branch it came from, for example `2 hours ago • feature/login`.
3. Pick a stash. It opens in a single editor titled `Git Stash #1: On feature/login: beta`, which shows every changed file, including untracked files if the stash has any.
4. While this editor is open, its title bar has **Apply Stash** and **Drop Stash** buttons. Use them to act on the stash you're looking at without picking it again.

#### Option C: Terminal

```bash
git stash list                       # all stashes with their messages
git stash list --date=relative       # add how long ago each was made
git stash show stash@{1}             # files changed in a stash
git stash show -p stash@{1}          # the full diff
git stash show --include-untracked stash@{1}   # include untracked files
```

---

## Step 4: Get a stash back (apply vs. pop)

| | Apply | Pop |
| --- | --- | --- |
| Restores the changes | Yes | Yes |
| Removes the stash from the list | No | Yes, but only if there are no conflicts |
| Use it when | You want to keep a copy, or apply the same changes on several branches | You're done with the stash |

Each comes in two forms. In the menu, both are under **More Actions (…) > Stash**.

- **Apply Latest Stash** and **Pop Latest Stash** act on `stash@{0}` right away, with no picker.
- **Apply Stash…** and **Pop Stash…** open the picker, so you can choose any stash by its message. **When you have more than one stash, use these.**

You can also apply or pop from the Stashes list (Step 3, Option A), or from the stash editor's title bar (Step 3, Option B).

### VS Code doesn't restore which changes were staged

VS Code's apply and pop restore your changes **without** Git's `--index` option. Tested result:

| Before stashing | After VS Code apply/pop | After `git stash apply --index` |
| --- | --- | --- |
| `f` had staged and unstaged edits | `f` modified, nothing staged | Staged and unstaged parts restored |
| `n` was a new, staged file | `n` staged (Git keeps new files staged) | `n` staged |

If you carefully staged part of your work and want it staged again, use Terminal:

```bash
git stash apply --index stash@{1}
git stash pop --index stash@{1}
```

### If there are conflicts

- If the stash conflicts with the current files, VS Code shows: "There are merge conflicts while applying the stash. Please resolve them before committing your changes." Click **Show Changes** to go to the conflicted files, then resolve them in the editor or the merge editor.
- **Pop with conflicts doesn't delete the stash.** Git says "The stash entry is kept in case you need it again." After you resolve the conflicts, drop the stash yourself (Step 5).
- If uncommitted changes to the same files would be overwritten, Git refuses to apply the stash. Commit or stash those changes first.

---

## Step 5: Delete stashes

- **One stash:** Run **Git: Drop Stash…** and pick it by message. Or right-click it in the Stashes list and choose **Drop Stash**, or click **Drop Stash** in the stash editor. VS Code asks "Are you sure you want to drop the stash: …?"
- **All stashes:** Run **Git: Drop All Stashes…**. VS Code warns that the stashes "MAY BE IMPOSSIBLE TO RECOVER" and asks you to confirm. This runs `git stash clear`.

Terminal equivalent:

```bash
git stash drop stash@{2}
git stash clear
```

**Recovering a dropped stash.** Stash commits stay in the repository until Git cleans them up, which usually takes a couple of weeks. To recover a dropped stash before then:

```bash
# Find stash commits that no longer have a reference
git fsck --no-reflog | awk '/dangling commit/ {print $3}' \
  | xargs git log --no-walk --format='%h %s' | grep -E 'WIP on|On '

# The list also includes stashes that are still in `git stash list`, so compare the two.
# Put the one you want back on the list
git stash store -m "recovered: login validation" <hash>
```

---

## Step 6: Working with many named stashes (Terminal recipes)

VS Code's pickers already let you choose by message. These Terminal commands help when you have a long list.

**Find a stash by name:**

```bash
git stash list --grep=login          # list only stashes whose message contains "login"
```

Don't use `stash^{/login}` for this. It only searches from the newest stash back through that stash's parent commits, so in testing it **didn't find** a stash that wasn't the newest one.

**Apply a stash by name** (a small helper for `~/.zshrc`):

```zsh
# Usage: sapply login        sapply login --index
sapply() {
  local ref
  ref=$(git log -g --format=%gd --grep-reflog="$1" refs/stash | head -1)
  [[ -z $ref ]] && { echo "No stash matching '$1'"; return 1; }
  echo "Applying $ref: $(git log -1 --format=%s "$ref")"
  git stash apply "${@:2}" "$ref"
}
```

**Rename a stash** (Git has no rename command):

```bash
h=$(git rev-parse stash@{1})
git stash drop stash@{1}
git stash store -m "login: validation, ready to review" "$h"
```

The renamed stash moves to `stash@{0}`, and it loses the `On <branch>:` prefix unless you type it into the new message.

**Turn a stash into a branch.** This is useful when a stash no longer applies cleanly:

```bash
git stash branch feature/login-retry stash@{1}
```

This creates a branch from the commit the stash was made on, applies the stash there, and drops the stash if that works.

---

## Stash prompts you'll see elsewhere in VS Code

- **When you check out a branch.** If uncommitted changes would be overwritten, VS Code shows "Your local changes would be overwritten by checkout." with three choices:
  - **Stash & Checkout** asks for a stash message, stashes your changes including untracked files, and checks out the branch. The changes **stay in the stash** until you apply them.
  - **Migrate Changes** does the same, then pops the stash on the new branch, so your changes come with you.
  - **Force Checkout** **discards** your changes.
- **When you pull.** With `git.autoStash: true`, VS Code runs `git pull --autostash`, which stashes your changes before the pull and restores them afterward. You don't need to do anything.

---

## Quick reference

| Task | VS Code (Command Palette) | Terminal |
| --- | --- | --- |
| Named stash | **Git: Stash** → type a message | `git stash push -m "msg"` |
| Include new files | **Git: Stash (Include Untracked)** | `git stash push -u -m "msg"` |
| Only staged changes | **Git: Stash Staged** | `git stash push -S -m "msg"` |
| Only some files | Stage them → **Git: Stash Staged** | `git stash push -m "msg" -- <paths>` |
| List | **Git: View Stash…** or the Stashes list | `git stash list` |
| Preview one | **Git: View Stash…** | `git stash show -p stash@{n}` |
| Apply and keep | **Git: Apply Stash…** | `git stash apply stash@{n}` |
| Apply and remove | **Git: Pop Stash…** | `git stash pop stash@{n}` |
| Keep staged parts staged | Not available | `git stash apply --index stash@{n}` |
| Newest only | **Git: Apply Latest Stash** / **Git: Pop Latest Stash** | `git stash apply` / `git stash pop` |
| Delete one | **Git: Drop Stash…** | `git stash drop stash@{n}` |
| Delete all | **Git: Drop All Stashes…** | `git stash clear` |
| Find by name | Type part of the message in the picker | `git stash list --grep=<text>` |
| Rename | Not available | `drop` + `store -m` (Step 6) |
| Stash → branch | Not available | `git stash branch <name> stash@{n}` |

---

## Troubleshooting

- **"There are no changes to stash."** Your only changes are untracked files. Use **Stash (Include Untracked)**.
- **An edit wasn't in the stash.** The file wasn't saved. Keep `git.promptToSaveFilesBeforeStash` at `"always"`.
- **Stash Staged is missing.** It needs Git 2.35 or later. Run `git --version`.
- **There's no Stashes list in Source Control.** Set `scm.repositories.selectionMode` to `"single"` and `scm.repositories.explorer` to `true`.
- **You applied the wrong stash.** If you used **Apply**, the stash is still on the list. Undo the applied changes with **Discard All Changes** in Source Control, but only if you had no other uncommitted work. Then apply the right one.
- **Your staged changes came back unstaged.** That's expected with VS Code's apply and pop. Use `git stash apply --index` (Step 4).
