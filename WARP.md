# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Overview

This is a ZMK (Zephyr Mechanical Keyboard) firmware configuration repository for a Lily58 split keyboard with Nice!Nano v2 controllers. The firmware is built automatically via GitHub Actions and supports wireless Bluetooth operation and experimental ZMK Studio live configuration editing.

## Key Architecture

### Build System
- **GitHub Actions**: Automated firmware builds triggered on push/PR/manual dispatch using `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`
- **Build Configuration**: `build.yaml` defines three build targets:
  - `lily58_left` with `studio-rpc-usb-uart` snippet (for ZMK Studio support)
  - `lily58_right`
  - `settings_reset` (for clearing keyboard memory)
- **West Manifest**: `config/west.yml` pulls ZMK from the official repository's main branch

### Firmware Configuration Files
- **config/lily58.keymap**: Main keymap file written in devicetree syntax
  - Contains 5 layers: default, right_hand_nums, left_hand_nums, vim, visual_vim
  - Extensive macro definitions for vim-like navigation, window management (Rectangle), IDE shortcuts, and system functions
  - Tap-dance behaviors for vim commands (dd, gg)
  - Conditional layers logic
- **config/lily58.conf**: Feature configuration flags
  - ZMK Studio enabled (`CONFIG_ZMK_STUDIO=y`)
  - Display enabled (`CONFIG_ZMK_DISPLAY=y`)
  - Battery percentage display enabled
  - Enhanced Bluetooth power (`CONFIG_BT_CTLR_TX_PWR_PLUS_8=y`)
  - Pointing device support enabled (`CONFIG_ZMK_POINTING=y`)

### Keymap Layer Structure
1. **Layer 0 (default)**: Standard QWERTY with modifiers
2. **Layer 1 (right_hand_nums)**: F-keys on left, numbers on right, arrow keys, Bluetooth pairing controls
3. **Layer 2 (left_hand_nums)**: Numbers on left, F-keys on right, macros (shortcat, screenshot, window management)
4. **Layer 3 (vim)**: Vim motion commands (hjkl, word navigation, delete, undo, gg/G)
5. **Layer 4 (visual_vim)**: Vim visual mode selections with shift modifiers

### Macros
The keymap defines numerous macros that combine modifier keys for:
- **MacOS Navigation**: Shortcat (`Cmd+Shift+Space`), window management with Rectangle app
- **IDE Operations**: IntelliJ shortcuts for file navigation and creation
- **Vim Emulation**: Motion commands (w/e/b), deletion (dd), navigation (gg/G), undo
- **System Functions**: Language toggle, screenshot selection, visual mode selections

## Common Commands

### Firmware Development
No local build commands - all builds happen in GitHub Actions. To test changes:
1. Push commits to trigger automatic build
2. Download firmware artifacts from the Actions tab once the green checkmark appears
3. Flash the `.uf2` files from the artifact to each keyboard half

### Flashing Firmware
```bash
# After downloading and extracting firmware.zip from GitHub Actions:
# 1. Connect left half, double-press RESET button
# 2. Copy *_left*.uf2 to the NICENANO drive
# 3. Connect right half, double-press RESET button  
# 4. Copy *_right*.uf2 to the NICENANO drive
```

### Reset Settings (if experiencing issues)
```bash
# Flash settings_reset.uf2 to both halves, then reflash normal firmware
```

### ZMK Studio Live Configuration
To unlock ZMK Studio for live editing:
- Press the layer 2 key (right half) + Esc simultaneously
- Access via https://zmk.studio in a Chromium-based browser
- Changes can be made over Bluetooth or USB without reflashing

## Editing Guidelines

### Modifying Keymaps
- Use devicetree syntax with proper indentation
- Key bindings use format: `&behavior KEY` (e.g., `&kp A`, `&mo 1`, `&trans`)
- Each layer must have exactly 58 key positions (Lily58 layout)
- Macros are defined in the `macros` block using `compatible = "zmk,behavior-macro"`
- Behaviors (like tap-dance) are defined in the `behaviors` block

### Adding New Macros
1. Define macro in the `macros` block with unique label
2. Use `<macro_press>`, `<macro_tap>`, `<macro_release>` sequences
3. Reference in keymap with `&macro_name`

### Layer Management
- `&mo N` = momentary layer N (hold)
- `&to N` = switch to layer N (persistent)
- `&tog N` = toggle layer N on/off
- `&trans` = pass through to lower layer
- Conditional layers defined in `conditional_layers` block

### Bluetooth Configuration
- Bluetooth selection keys: `&bt BT_SEL 0` through `&bt BT_SEL 4` (layer 1)
- Layer 1 accessible by holding the key at position [4,2] (the "LOWER" key on default layer)

### Configuration Changes
Edit `config/lily58.conf` to enable/disable features:
- Display, encoder, Bluetooth power, battery indicators
- ZMK Studio support (currently enabled)
- Pointing device support (currently enabled)

## Important Notes
- This configuration is in Russian - keymap comments and README use Cyrillic
- Macros are heavily customized for MacOS (Cmd/Alt modifiers)
- The keyboard has vim emulation layers designed for text editing without leaving home row
- Never commit firmware files (`.uf2`) - they're build artifacts
- ZMK Studio support is experimental and requires the `studio-rpc-usb-uart` snippet on the left half
