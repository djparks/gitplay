# Yes. Each worktree is its own folder and opens in its own window, so each one can have its own colors. You set them in a per-folder settings file. This setup works in VS Code 1.148 on macOS, and the title bar colors work with the default title bar style

## 1. Add colors to each worktree's folder settings

Create `.vscode/settings.json` inside each worktree folder. Leave the main folder without one so it keeps your normal theme.

`myapp.worktrees/feature-login/.vscode/settings.json` (green):

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

`myapp.worktrees/hotfix-crash/.vscode/settings.json` (yellow, with dark text so it stays readable):

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

The window updates as soon as you save; you don't need to reload. If you only want a hint of color, keep just the `statusBar.*` lines.

## 2. Keep the color file out of commits

If the file got committed, the color would follow the branch: merging `feature-login` would turn `main` green. How you avoid that depends on whether your repository already tracks `.vscode/settings.json`. This repo doesn't track it.

- **The repo doesn't track it:** add `.vscode/settings.json` to `.git/info/exclude`. That file works like `.gitignore` but is never committed, and every worktree of the repository uses it, so you only add the line once.
- **The repo tracks it (for shared formatter settings, say):** add the color block to that worktree's copy, then run `git update-index --skip-worktree .vscode/settings.json` inside that worktree. Git then ignores your local changes to that file in that worktree only.

## 3. Optional: set the color when you create a worktree

You could extend the `wt` helper from the worktrees guide to take a color, for example `wt feature/login 2e7d32`, and write the settings file for you. The **Peacock** extension does the same thing from a color picker in VS Code. It writes these same settings into the folder's settings file, so step 2 still applies.

## Limitation

Colors apply to a whole window. In a multi-root workspace with several worktrees in one window (Option B in Part 2 of the worktrees guide), you can't give each folder its own color. Use one window per worktree if you want them color-coded.

Do you want me to add this as a section in the worktrees guide, along with the `wt` color option?
