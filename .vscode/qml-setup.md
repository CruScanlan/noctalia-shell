# QML Language Server Setup for Noctalia (NixOS)

This document explains the QML language server configuration for the Noctalia Shell project on NixOS.

## What Was Configured

### VSCode Settings (`.vscode/settings.json`)
- Added basic import path configuration
- The language server provides syntax highlighting and basic Qt/Quickshell type support

### qmlls Binary Symlink
- Replaced the extension's downloaded binary with a Nix-compatible version
- Location: `~/.local/share/qmlls/files/qmlls` → symlinks to Nix store qmlls

## Important: No qmldir Files

**We cannot use `qmldir` files** with Noctalia/Quickshell because:
- Quickshell uses a custom QML import system
- Standard QML `qmldir` files break Quickshell's module resolution
- This causes runtime errors like "Property 'i' of object Logger is not a function"

## What Works

✅ **Syntax highlighting** for QML
✅ **Basic autocomplete** for Qt/Quickshell types  
✅ **Error checking** for syntax errors
✅ **Quickshell runtime** works perfectly

## What Doesn't Work

❌ **Full type resolution** for custom `qs.*` imports (requires qmldir files)
❌ **Go-to-definition** for custom modules
❌ **Autocomplete** for custom types like `Logger`, `Settings`, etc.

This is a known limitation - you must choose between perfect IDE support OR a working Quickshell runtime. We chose the working runtime.

## How to Use

The language server should work automatically once the symlink is in place. Just reload the window:
- Press `Ctrl+Shift+P` → "Developer: Reload Window"

## What to Expect

You'll get basic QML language support:
- Syntax errors will be highlighted
- Qt and Quickshell built-in types will autocomplete
- Your custom `qs.*` imports will NOT show errors (they'll just be unresolved)

## NixOS-Specific Fix

The Qt QML extension tries to download its own `qmlls` binary from GitHub, but this won't work on NixOS because:
- NixOS can't run dynamically linked executables built for standard Linux
- The error is: "Could not start dynamically linked executable"

**Solution**: We replaced the downloaded binary with a symlink to the Nix-provided qmlls:

```bash
# Backup the downloaded binary
mv ~/.local/share/qmlls/files/qmlls ~/.local/share/qmlls/files/qmlls.backup

# Find the Nix qmlls
QMLLS_PATH=$(find /nix/store -name "qmlls" 2>/dev/null | grep qtdeclarative | head -1)

# Create symlink
ln -s "$QMLLS_PATH" ~/.local/share/qmlls/files/qmlls
```

This way, the extension finds `qmlls` where it expects it, but it's actually using the NixOS-compatible version.

## Troubleshooting

If it's still not working:

1. **Check the symlink**: Run `ls -la ~/.local/share/qmlls/files/qmlls` - it should be a symlink to a Nix store path

2. **Test qmlls directly**: Run `~/.local/share/qmlls/files/qmlls --version` - it should print the version

3. **Check the language server is running**: Look for "QML Language Server" in the Output panel (View → Output, then select "qt-qml" from the dropdown)

4. **After updating Qt**: If you update your Qt/qtdeclarative package, you'll need to update the symlink:
   ```bash
   rm ~/.local/share/qmlls/files/qmlls
   ln -s $(find /nix/store -name "qmlls" 2>/dev/null | grep qtdeclarative | head -1) ~/.local/share/qmlls/files/qmlls
   ```

## Note on qmldir Files

**DO NOT add qmldir files to this project!** They will break Quickshell at runtime.

If you see `qmldir` files in any directory, delete them immediately:
```bash
find . -name "qmldir" -type f -delete
```




