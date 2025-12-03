# Noctalia Shell - Local Development Guide

This guide explains how to set up live development for Noctalia Shell with hot-reloading capabilities.

## Prerequisites

- NixOS or Nix package manager
- Noctalia Shell repository cloned locally
- Quickshell installed

## Setup Live Development

### Step 1: Create Symlink

Link your development repository to the quickshell config directory:

```bash
# Remove existing noctalia-shell config (if any)
rm -rf ~/.config/quickshell/noctalia-shell

# Create symlink to your dev repo
ln -s /path/to/your/noctalia-shell ~/.config/quickshell/noctalia-shell
```

For example:
```bash
ln -s /home/cru/projects/noctalia-shell ~/.config/quickshell/noctalia-shell
```

### Step 2: Build Helper Scripts

From the noctalia-shell repository:

```bash
cd /path/to/your/noctalia-shell
nix build .#noctalia-live-devel
nix build .#qs-noctalia
```

### Step 3: Launch Development Server

**Option A: Run from build result**
```bash
./result/bin/noctalia-live-devel
```

**Option B: Use nix shell (temporary)**
```bash
nix shell .#noctalia-live-devel .#qs-noctalia
noctalia-live-devel
```

**Option C: Install permanently (add to your NixOS configuration)**
```nix
{
  environment.systemPackages = [
    inputs.noctalia-shell.packages.${system}.noctalia-live-devel
    inputs.noctalia-shell.packages.${system}.qs-noctalia
  ];
}
```

Then simply run:
```bash
noctalia-live-devel
```

## What Happens When You Launch

1. The noctalia-shell systemd service is stopped (if running)
2. Quickshell starts with your development directory
3. **Live reloading is enabled** - any QML file changes are instantly reflected
4. When you exit (Ctrl+C), the systemd service automatically restarts

## Using qs-noctalia

The `qs-noctalia` helper is a drop-in replacement for `noctalia-shell` that:
- Detects if the dev server is running
- Routes IPC commands to the dev server OR the regular service automatically

Example:
```bash
qs-noctalia ipc
```

## Development Workflow

1. Launch the dev server: `./result/bin/noctalia-live-devel`
2. Edit QML files in your repository
3. Changes appear instantly in the shell
4. Test your changes
5. Exit with `Ctrl+C` when done (systemd service restarts automatically)

## Cleanup / Removal

### Remove Development Setup

**Step 1: Stop the dev server**
```bash
# Press Ctrl+C in the terminal running noctalia-live-devel
```

**Step 2: Remove the symlink**
```bash
rm ~/.config/quickshell/noctalia-shell
```

**Step 3: Restore normal configuration**

If you want to restore the regular noctalia-shell config:
```bash
# Option A: Let noctalia-shell systemd service recreate it
systemctl restart --user noctalia-shell

# Option B: Manually restore from package
# The exact path depends on your NixOS configuration
```

**Step 4: Clean up build artifacts (optional)**
```bash
cd /path/to/your/noctalia-shell
rm result  # Remove the nix build symlink
```

**Step 5: Remove helper packages (if installed system-wide)**

If you added the packages to your NixOS configuration, remove:
```nix
# Remove these lines from your configuration.nix
inputs.noctalia-shell.packages.${system}.noctalia-live-devel
inputs.noctalia-shell.packages.${system}.qs-noctalia
```

Then rebuild:
```bash
sudo nixos-rebuild switch
```

## Troubleshooting

### "Permission denied" errors
Some template files may have permission issues. These are usually cosmetic and don't affect core functionality.

### Service not stopping
If the systemd service doesn't exist or isn't running:
```bash
systemctl status --user noctalia-shell
```

The dev server will still work; you'll just see a warning message.

### Changes not appearing
1. Check that the symlink is correct: `readlink -f ~/.config/quickshell/noctalia-shell`
2. Ensure you're editing files in the symlinked directory
3. Check quickshell logs for errors: `journalctl --user -u noctalia-shell -f`

### Restore working state quickly
```bash
# Stop dev server (Ctrl+C)
rm ~/.config/quickshell/noctalia-shell
systemctl restart --user noctalia-shell
```

## Tips

- Keep the terminal with the dev server visible to see logs in real-time
- QML errors will appear immediately in the logs
- You can launch multiple dev servers for different configurations
- Use `jj` or `git` to manage your changes separately from the live dev setup

## More Information

For more details, see [PR #950](https://github.com/noctalia-dev/noctalia-shell/pull/950)

