# VS Code Working Sets on macOS: Per-Branch Editors, Cursors, and Split Layouts

This guide configures VS Code to remember your open tabs, editor splits, and cursor positions for each Git branch. When you switch branches, VS Code saves the workspace you're leaving and restores the one for the branch you're switching to.

## What the feature does

- **Settings:** `scm.workingSets.enabled` (boolean, default `false`) and `scm.workingSets.default` (`"current"` or `"empty"`, default `"current"`). Both arrived in [VS Code 1.89 (April 2024)](https://code.visualstudio.com/updates/v1_89).
- **When it saves:** At the moment VS Code sees the branch change. It saves a snapshot of the branch you're leaving and then applies the snapshot for the branch you're switching to ([source: `workingSet.ts`](https://raw.githubusercontent.com/microsoft/vscode/main/src/vs/workbench/contrib/scm/browser/workingSet.ts)).
- **What the snapshot holds:** The full editor-part state for the main window and any floating windows. That covers the split grid (orientation, nesting, sizes), each editor group's tabs, pinned and sticky tabs, and the active tab ([source: `editorParts.ts`](https://raw.githubusercontent.com/microsoft/vscode/main/src/vs/workbench/browser/parts/editor/editorParts.ts)).
- **Cursor and scroll positions:** These aren't stored in the working set. VS Code reapplies them through its per-file view-state memory when an editor reopens, which is controlled by `workbench.editor.restoreViewState` (on by default). Step 3 makes sure that setting stays on.
- **How the setting describes itself:** "Controls whether to store editor working sets when switching between source control history item groups," meaning branches ([source: `scm.contribution.ts`](https://github.com/microsoft/vscode/blob/8fb60541/src/vs/workbench/contrib/scm/browser/scm.contribution.ts)).

## Prerequisites

1. **Check your version.** VS Code must be 1.89 or later. Go to **Code > About Visual Studio Code**.
2. **Make sure Git is available.** Run `git --version` in Terminal. If Git is missing, run `xcode-select --install`.
3. **Install the `code` command (optional, used in later steps).** Press `Cmd+Shift+P`, then run **Shell Command: Install 'code' command in PATH**.
4. **Open the repository as a folder or workspace.** Use **File > Open Folder…**, not individual files, because working sets are stored per workspace.

## Step 1: Enable working sets

### Option A: Settings UI

1. Press `Cmd+,`.
2. Type `scm.workingSets` in the search box.
3. Check **Scm > Working Sets: Enabled**.
4. Set **Scm > Working Sets: Default** to the value you want (see Step 2).
5. Choose the **User** tab to apply this to every repository, or the **Workspace** tab to apply it to this repository only.

### Option B: settings.json

Press `Cmd+Shift+P` and run **Preferences: Open User Settings (JSON)**. On macOS, this file is located at:

```
~/Library/Application Support/Code/User/settings.json
```

(For VS Code Insiders, replace `Code` with `Code - Insiders`.)

Add the following:

```jsonc
{
  // Save and restore editors and split layout per Git branch
  "scm.workingSets.enabled": true,

  // What to show the first time you switch to a branch that has no saved set:
  //   "current" = keep the editors you have open now (default)
  //   "empty"   = start with no editors open
  "scm.workingSets.default": "empty"
}
```

For a single repository, put the same keys in `<repo>/.vscode/settings.json` instead.

## Step 2: Choose a default for new branches

| Value | Behavior on first switch to a branch | Best for |
|---|---|---|
| `current` | Keeps the tabs and layout you already have open | Branching from `main` and continuing on the same files |
| `empty` | Closes all editors so you start clean | Keeping contexts separate across feature, bugfix, and review branches |

Once a branch has a saved snapshot, this setting has no further effect on that branch.

## Step 3: Keep cursor and scroll positions

Add these settings to the same `settings.json` so reopened files land on your last cursor position and the overall layout survives restarts:

```jsonc
{
  // Restore cursor, selection, scroll, and folding when a file reopens (default: true)
  "workbench.editor.restoreViewState": true,

  // Restore the window and its editors when you quit and relaunch VS Code
  "window.restoreWindows": "all",

  // Optional: save files automatically so a branch switch never stops on a dirty-file prompt
  "files.autoSave": "onFocusChange"
}
```

`restoreViewState` is on by default, so you only need this if you or an extension turned it off.

## Step 4: Build a per-branch layout

1. **Start on `main`.** Open the files you want and arrange your splits:
   - `Cmd+\` splits the editor to the right.
   - `Cmd+K Cmd+\` splits down (or use **View > Editor Layout**).
   - `Cmd+K Shift+Enter` pins a tab.
   - Put the cursor where you want it in each file.
2. **Switch branches** in any of these ways:
   - Click the branch name in the status bar (bottom left).
   - Press `Cmd+Shift+P` and run **Git: Checkout to…**.
   - Run `git switch feature/x` in the integrated terminal. Working sets also respond to terminal switches.
3. **On the new branch,** open and arrange that branch's files.
4. **Switch back to `main`.** Your earlier tabs, splits, and cursor positions come back.

## Step 5: Verify it's working

1. **Test a round trip.** Open two files side by side on `main`, switch to another branch, then switch back. The split should reappear.
2. **Inspect the stored state (optional).** Working sets live in the workspace's local storage database, under the keys `scm.workingSets` and `editor.workingSets` ([source](https://raw.githubusercontent.com/microsoft/vscode/main/src/vs/workbench/contrib/scm/browser/workingSet.ts)). Find the database:

   ```bash
   # Locate the workspace storage folder for your repo
   grep -l "$(pwd)" ~/Library/Application\ Support/Code/User/workspaceStorage/*/workspace.json

   # Inspect it (macOS ships with sqlite3). Quit VS Code first to avoid lock issues.
   sqlite3 ~/Library/Application\ Support/Code/User/workspaceStorage/<HASH>/state.vscdb \
     "SELECT key, length(value) FROM ItemTable WHERE key LIKE '%workingSets%';"
   ```

   A row for `scm.workingSets` with a non-zero length means snapshots are being saved.

## Recommended full configuration

```jsonc
{
  "scm.workingSets.enabled": true,
  "scm.workingSets.default": "empty",
  "workbench.editor.restoreViewState": true,
  "window.restoreWindows": "all",
  "files.autoSave": "onFocusChange",
  "git.autofetch": true
}
```

## Limitations and troubleshooting

- **Unsaved changes.** If a tab has unsaved edits, a branch switch can stop on a save prompt or fail the checkout. Save first, or turn on auto-save.
- **Files that exist only on one branch.** Earlier versions had a [reported bug](https://github.com/microsoft/vscode/issues/212327) where branch-only files didn't reopen. If you see this, update VS Code.
- **Terminal focus.** Switching branches in the terminal used to [move focus into an editor](https://github.com/microsoft/vscode/issues/212048). Current code keeps focus in the panel, but not when the terminal is opened as an editor tab ([source](https://raw.githubusercontent.com/microsoft/vscode/main/src/vs/workbench/contrib/scm/browser/workingSet.ts)).
- **Extension webviews.** Tabs such as Git Graph [may close on a branch switch](https://github.com/mhutchie/vscode-git-graph/issues/823) because they can't always be serialized.
- **Turning the setting off erases saved sets.** Disabling `scm.workingSets.enabled` deletes the stored `scm.workingSets` data for the workspace ([source](https://raw.githubusercontent.com/microsoft/vscode/main/src/vs/workbench/contrib/scm/browser/workingSet.ts)). Don't toggle it off and on to troubleshoot unless you're fine losing them.
- **No sync.** Snapshots are stored per machine and per workspace (`StorageTarget.MACHINE`), so Settings Sync doesn't carry them to another Mac. Your settings do sync.
- **Git worktrees.** Each worktree folder is a separate workspace with its own storage. Worktrees are a good alternative if you want branches open side by side ([VS Code docs](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)).
- **Cursor and other forks.** Forks can lag behind. One Cursor user reported [working sets no longer function](https://forum.cursor.com/t/working-sets-no-longer-function/157330) there.
- **Detached HEAD.** Snapshots are keyed to the current branch reference. Checking out a raw commit or tag may not map to the branch you expect.
