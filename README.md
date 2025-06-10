# VS Code Configuration

This repository contains my personal Visual Studio Code configuration files.

## Files

- `settings.json`: Editor and workspace settings.
- `extensions.txt`: List of installed extensions.
- `keybindings.json`: Custom keyboard shortcuts.

## How to Generate These Files

### Exporting Settings

1. Open VS Code.
2. Go to `Code` > `Preferences` > `Settings` (or press `Cmd + ,` on Mac / `Ctrl + ,` on Windows/Linux).
3. Click the `{}` icon in the top right to open `settings.json`.
4. Copy the contents and save them as `settings.json` in this repository.

### Exporting Extensions List

1. Open a terminal in VS Code.
2. Run the following command to list all installed extensions:

   ```sh
   code --list-extensions > extensions.txt
   ```

3. Copy or move the generated `extensions.txt` to this repository.

### Exporting Keybindings

1. Open VS Code.
2. Go to `Code` > `Preferences` > `Keyboard Shortcuts` (or press `Cmd + K Cmd + S`).
3. Click the `{}` icon in the upper right to open `keybindings.json`.
4. Copy the contents and save them as `keybindings.json` in this repository.

## Usage

To replicate this setup on a new machine:

1. Copy `settings.json` to your VS Code user settings folder.
2. Copy `keybindings.json` to your VS Code keybindings folder.
3. Install all extensions from `extensions.txt`:

   ```sh
   cat extensions.txt | xargs -L 1 code --install-extension
   ```

---

Feel free to customize these files to fit your workflow.
