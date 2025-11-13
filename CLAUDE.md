# CLAUDE.md - Claude Code Integration Guide

This file describes the repository structure, conventions, and best practices for working with this ZMK keyboard firmware project using Claude Code.

## Project Overview

**Repository**: Sofle Hybrid Ergomech ZMK Configuration
**Type**: Split wireless ergonomic keyboard firmware (ZMK-based)
**Language**: Device Tree Source (DTS), Keymap definition (ZMK), YAML, Markdown
**Primary Focus**: User keymap customization and firmware configuration

## Repository Structure

```
.
├── config/                               # USER-EDITABLE - Your custom configuration
│   ├── sofle_ergomech.keymap            # ⭐ Main file to edit - your custom keymap
│   ├── sofle_ergomech.conf              # Feature toggles (display, RGB, etc.)
│   ├── sofle_ergomech.json              # Keyboard layout metadata for tools
│   ├── info.json                        # Keyboard information
│   └── west.yml                         # Dependency manifest
│
├── boards/shields/sofle_ergomech/       # HARDWARE DEFINITIONS - Read-only (mostly)
│   ├── sofle_ergomech.dtsi              # Main hardware definition
│   ├── sofle_ergomech-layouts.dtsi      # Layout configuration
│   ├── sofle_ergomech.keymap            # Default keymap reference
│   ├── sofle_ergomech.conf              # Default configuration
│   ├── sofle_ergomech_left.overlay      # Left half configuration
│   ├── sofle_ergomech_right.overlay     # Right half configuration
│   ├── sofle_ergomech_left.conf         # Left half specific config
│   ├── sofle_ergomech_right.conf        # Right half specific config
│   ├── Kconfig.shield                   # Kconfig definitions
│   ├── Kconfig.defconfig                # Default configurations
│   ├── sofle_ergomech.zmk.yml           # ZMK Studio metadata
│   └── boards/                          # Microcontroller-specific overlays
│       ├── nice_nano_v2.overlay
│       ├── nice_nano.overlay
│       ├── nrfmicro_13.overlay
│       └── nrfmicro_11.overlay
│
├── .github/workflows/                   # CI/CD Automation
│   ├── build.yml                        # Main firmware build pipeline
│   └── keymap_drawer.yaml               # Visual keymap generation
│
├── keymap-drawer/                       # Keymap visualization files
│   ├── sofle_ergomech.yaml              # Keymap drawer configuration
│   └── sofle_ergomech.svg               # Generated keymap visualization
│
├── zephyr/
│   └── module.yml                       # Zephyr module configuration
│
├── build.yaml                           # GitHub Actions build matrix
├── KEYMAP_GUIDELINES.md                 # User guidelines (comprehensive)
├── CLAUDE.md                            # This file
├── README.md                            # Project documentation
├── keymap.svg                           # Visual keymap output
├── sofle_ergomech.yaml                  # Keymap drawer config
└── .gitignore
```

## Key Files for Customization

### 1. **config/sofle_ergomech.keymap** ⭐ PRIMARY EDIT FILE

This is the main file you'll edit to customize your keyboard layout.

**Format**: ZMK keymap definition
**Purpose**: Define which key is at which position across all layers
**Structure**: Layers > rows > key bindings

**Example modification**:
```c
keymap {
    compatible = "zmk,keymap";

    default_layer {
        label = "Default";
        bindings = <
            &kp ESC    &kp N1     &kp N2     // Row 0
            &kp TAB    &kp Q      &kp W      // Row 1
            // ... more rows
        >;
    };

    nav_layer {
        label = "Navigation";
        bindings = <
            &kp GRAVE  &none      &none      // Row 0
            &trans     &kp HOME   &kp UP     // Row 1
            // ... more rows
        >;
    };
};
```

### 2. **config/sofle_ergomech.conf**

Configuration flags for enabling/disabling features.

**Common options**:
```conf
CONFIG_ZMK_DISPLAY=y              # Enable OLED display
CONFIG_EC11=y                     # Enable encoder support
CONFIG_ZMK_RGB_UNDERGLOW=y        # Enable RGB LEDs
CONFIG_ZMK_WIRELESS=y             # Enable Bluetooth
CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START=y  # Clear pairings on startup
```

### 3. **boards/shields/sofle_ergomech/sofle_ergomech.dtsi**

Main hardware definition. Generally don't edit unless:
- Adding new keys to the layout
- Changing pin assignments
- Modifying display/RGB configuration

## Common Tasks

### Task: Customize Your Keymap

1. **Edit file**: `config/sofle_ergomech.keymap`
2. **Modify key codes** in the `bindings` array
3. **Add/remove layers** by creating new layer definitions
4. **Commit**: `git add config/sofle_ergomech.keymap && git commit -m "custom keymap"`
5. **Build**: Push to GitHub, GitHub Actions builds automatically
6. **Flash**: Download `.uf2` files and flash to both halves

**Key concepts**:
- `&kp KEY_CODE` - Press a key
- `&mo LAYER` - Momentary layer (hold)
- `&lt LAYER KEY` - Layer-tap (hold=layer, tap=key)
- `&tog LAYER` - Toggle layer
- `&trans` - Transparent (pass-through to lower layer)
- `&none` - Do nothing

### Task: Enable/Disable Features

1. **Edit file**: `config/sofle_ergomech.conf`
2. **Add/remove configuration flags**:
   - Display: `CONFIG_ZMK_DISPLAY=y`
   - RGB: `CONFIG_ZMK_RGB_UNDERGLOW=y`
   - Encoder: `CONFIG_EC11=y`
3. **Commit and push**
4. **GitHub Actions rebuilds** with new features

### Task: Add a Macro

1. **Edit file**: `config/sofle_ergomech.keymap`
2. **Add behavior definition**:
```c
behaviors {
    hello: hello_macro {
        compatible = "zmk,behavior-macro";
        label = "HELLO_MACRO";
        #binding-cells = <0>;
        bindings = <&kp H &kp E &kp L &kp L &kp O>;
    };
};
```
3. **Use in keymap**: `&hello`

### Task: Add a Combo

1. **Edit file**: `config/sofle_ergomech.keymap`
2. **Add combo definition**:
```c
combos {
    compatible = "zmk,combos";

    combo_esc {
        timeout-ms = <50>;
        key-positions = <0 1>;  // First two keys together
        bindings = <&kp ESC>;
    };
};
```
3. **Rebuild firmware**

### Task: Adjust RGB/Display Configuration

1. **Edit file**: `boards/shields/sofle_ergomech/sofle_ergomech.dtsi`
2. **Modify LED/display sections** if needed
3. **Common changes**:
   - LED brightness: `CONFIG_ZMK_LED_LEVEL_DEFAULT`
   - Display timeout: `timeout-ms` property

## ZMK Key Codes & Syntax

### Basic Key Syntax

```c
&kp KEY_CODE              // Press a key
&kp LSHIFT A              // Modifier + key (capital A)
&kp LCTRL A               // Ctrl+A
```

### Layer Control

```c
&mo 1                     // Momentary: hold for layer 1
&lt 1 SPACE               // Layer-tap: hold=layer1, tap=space
&tog 1                    // Toggle: press to switch layer 1 on/off
&trans                    // Transparent (use key from lower layer)
&none                     // No action
```

### Special Keys

```c
// Modifiers
LSHIFT, RSHIFT, LCTRL, RCTRL, LALT, RALT, LGUI, RGUI

// Navigation
UP, DOWN, LEFT, RIGHT, HOME, END, PGUP, PGDN

// Special
SPACE, ENTER, BSPC, DEL, TAB, ESC
GRAVE, MINUS, EQUAL, LBKT, RBKT, BSLH
SEMI, SQT, COMMA, DOT, FSLH

// Function
F1, F2, ... F12

// Media
C_VOL_UP, C_VOL_DN, C_MUTE, C_PLAY_PAUSE, C_NEXT, C_PREV
C_BRI_UP, C_BRI_DN
```

## Build & Flash Workflow

### Automated (GitHub Actions)

1. **Edit** `config/sofle_ergomech.keymap`
2. **Commit**: `git add . && git commit -m "description"`
3. **Push**: `git push`
4. **Wait**: GitHub Actions builds (see Actions tab)
5. **Download**: Get `.uf2` files from artifacts
6. **Flash**: Double-tap reset on each half, drag file onto USB drive

### Manual Build (Local)

Requires ZMK tools installed:

```bash
# Update dependencies
west update

# Build right half
west build -b nice_nano_v2 -d build/right -- -DSHIELD=sofle_ergomech_right

# Build left half
west build -b nice_nano_v2 -d build/left -- -DSHIELD=sofle_ergomech_left

# Find .uf2 files in build/right/zephyr and build/left/zephyr
```

## Important Notes for Claude Code

### What to Edit
- ✅ `config/sofle_ergomech.keymap` - Your keymap
- ✅ `config/sofle_ergomech.conf` - Feature configuration
- ✅ `.github/workflows/` - If modifying build process
- ✅ Documentation files (`.md`)

### What NOT to Edit (Usually)
- ❌ `boards/shields/sofle_ergomech/sofle_ergomech.dtsi` - Hardware definition (unless deep changes)
- ❌ `boards/shields/sofle_ergomech/boards/*.overlay` - Microcontroller configs
- ❌ File structure under `boards/` - Can break hardware detection

### When to Suggest Full Rebuild
- After editing `.dtsi` files
- After changing `CONFIG_ZMK_*` flags
- After modifying hardware definitions
- After adding new behaviors or combos

### Flashing Considerations
- **Split keyboard**: Must flash BOTH halves separately
- **Reset button**: Double-tap to enter bootloader mode
- **USB drive**: Keyboard appears as USB storage when in bootloader
- **File drag-drop**: Simply drag the `.uf2` file onto the USB drive
- **Test in stages**: Flash one change at a time, test before next change

## Matrix Layout Reference

### Sofle Hybrid Ergomech Matrix

**Left Half** (Columns 0-5):
```
Col:  0    1    2    3    4    5
Row0: KC00 KC01 KC02 KC03 KC04 KC05
Row1: KC10 KC11 KC12 KC13 KC14 KC15
Row2: KC20 KC21 KC22 KC23 KC24 KC25
Row3: KC30 KC31 KC32 KC33 KC34 KC35
Row4: KC40 KC41 KC42 (Thumb cluster)
```

**Right Half** (Columns 6-11):
```
Col:  6    7    8    9   10   11
Row0: KC06 KC07 KC08 KC09 KC10 KC11
Row1: KC16 KC17 KC18 KC19 KC20 KC21
Row2: KC26 KC27 KC28 KC29 KC30 KC31
Row3: KC36 KC37 KC38 KC39 KC40 KC41
Row4: KC46 KC47 KC48 KC49 KC50 KC51 (5-way switch)
```

Key position reference: `RC(row, col)` or in bindings just position in sequence

## Version Control Best Practices

### Commits

```bash
# Good commit message
git commit -m "custom keymap: add navigation layer and media controls"
git commit -m "config: enable RGB underglow with 30% brightness"

# Avoid generic messages
git commit -m "update"  # ❌ Not helpful
```

### Branches

- Main branch: `sofle_hybrid_rgb_rev5`
- Feature branches for major changes (optional): `feature/gaming-layer`, `fix/rgb-config`

### Push & PR

```bash
git add .
git commit -m "description"
git push origin sofle_hybrid_rgb_rev5
```

For major changes, create a PR on GitHub for review before merging.

## Troubleshooting Common Issues

### Problem: Changes not taking effect after flashing

**Solutions**:
1. Ensure changes were pushed to GitHub (check repo)
2. Wait for GitHub Actions to finish building
3. Download the LATEST `.uf2` files (check timestamps)
4. Flash BOTH halves (left AND right)
5. Try a full settings reset: flash `settings_reset-nice_nano_v2-zmk.uf2` to both halves

### Problem: Keys are in wrong positions

**Solutions**:
1. Check matrix coordinates (left=0-5, right=6-11)
2. Verify you edited the correct layer
3. Compare your bindings with default keymap
4. Ensure no typos in key codes

### Problem: Bluetooth won't pair

**Solutions**:
1. Factory reset both halves with `settings_reset` file
2. Re-pair in system Bluetooth settings
3. Ensure both halves are powered on
4. Check `CONFIG_ZMK_WIRELESS=y` is enabled

### Problem: Display/RGB not working

**Solutions**:
1. Verify feature is enabled in `config/sofle_ergomech.conf`
2. Check hardware connections (solder joints)
3. Try `settings_reset` to clear any corrupt state

## Integration with Claude Code Workflows

### Workflow 1: Keymap Customization

```bash
# User describes desired keymap → Claude suggests modifications
# → User approves → Claude edits config/sofle_ergomech.keymap
# → Claude commits with meaningful message
# → GitHub Actions builds → Artifacts available for download
```

### Workflow 2: Feature Enablement

```bash
# User wants to enable a feature (e.g., RGB)
# → Claude edits config/sofle_ergomech.conf
# → Claude commits and explains what changed
# → GitHub Actions rebuilds with feature
```

### Workflow 3: Troubleshooting

```bash
# User reports an issue (e.g., key in wrong place)
# → Claude examines the keymap file
# → Claude identifies the problem and suggests fix
# → Claude makes the fix, commits, and re-builds
```

## Resources & Documentation

- [ZMK Official Docs](https://zmk.dev) - Comprehensive firmware documentation
- [ZMK Keycodes](https://zmk.dev/docs/codes/keyboard-keymap) - All available key codes
- [ZMK Behaviors](https://zmk.dev/docs/behaviors/overview) - Layers, macros, combos, etc.
- [Nice!Nano Docs](https://nicekeyboards.com) - Microcontroller documentation
- [Sofle Original Project](https://github.com/soflettkydid/Sofle) - Original keyboard design
- [KEYMAP_GUIDELINES.md](./KEYMAP_GUIDELINES.md) - Comprehensive user guide in this repo

## Quick Reference Commands

```bash
# View recent commits
git log --oneline -10

# Check current status
git status

# Commit and push
git add . && git commit -m "message" && git push

# View GitHub Actions builds
# Visit: https://github.com/YOUR_USERNAME/sofle-hybrid-ergomech-zmk/actions

# Check git diff before committing
git diff config/sofle_ergomech.keymap
```

## When to Ask User for Clarification

- What is their primary use case for the keyboard?
- Do they prefer certain keys in specific locations?
- What layers do they want to set up?
- How many keys should be reassigned?
- Do they want macros or complex behaviors?
- What microcontroller are they using? (usually Nice!Nano v2)
- Which side is left, which is right in their setup?

## Success Metrics

User has successfully completed a task when:

1. ✅ Keymap is edited as requested
2. ✅ Changes are committed with clear message
3. ✅ Changes are pushed to GitHub
4. ✅ GitHub Actions completes build
5. ✅ User flashes both halves
6. ✅ Keyboard functionality matches request
7. ✅ User confirms satisfaction

---

**Last Updated**: 2025-11-13
**Firmware**: ZMK
**Keyboard**: Sofle Hybrid Ergomech Rev 5
**Target Audience**: Claude Code + User collaboration
