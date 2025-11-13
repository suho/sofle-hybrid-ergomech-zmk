# Sofle Hybrid Ergomech ZMK - Keymap Guidelines

A comprehensive guide for customizing your Sofle Hybrid Ergomech Rev 5 keyboard with ZMK firmware.

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Understanding Your Keyboard](#understanding-your-keyboard)
3. [Keymap Basics](#keymap-basics)
4. [Creating a Custom Keymap](#creating-a-custom-keymap)
5. [ZMK Key Codes Reference](#zmk-key-codes-reference)
6. [Advanced Features](#advanced-features)
7. [Flashing Your Keyboard](#flashing-your-keyboard)
8. [Troubleshooting](#troubleshooting)

---

## Repository Overview

This is the **Sofle Hybrid Ergomech Rev 5** - a professional split wireless ergonomic keyboard powered by ZMK (Zephyr Mechanical Keyboard).

### Key Specifications

| Aspect | Details |
|--------|---------|
| **Type** | Split wireless ergonomic keyboard |
| **Layout** | 6×4 main keys + 5-key thumb cluster per side, column-staggered |
| **Switches** | Kailh Choc (low profile) |
| **Microcontroller** | Nice!Nano v2 (nRF52840) |
| **Connectivity** | Bluetooth 5.0 + USB-C wired |
| **Features** | OLED display, RGB underglow (36 LEDs/side), 5-way directional switch |
| **Firmware** | ZMK - Open source, Bluetooth-native |

### Project Structure

```
sofle-hybrid-ergomech-zmk/
├── config/                          # User-editable files
│   ├── sofle_ergomech.keymap        # Main keymap (EDIT THIS)
│   ├── sofle_ergomech.conf          # Feature configuration
│   ├── sofle_ergomech.json          # Keyboard layout metadata
│   └── info.json                    # Keyboard information
├── boards/shields/sofle_ergomech/   # Hardware definitions (don't edit)
│   ├── sofle_ergomech.dtsi          # Main hardware config
│   ├── sofle_ergomech.keymap        # Default keymap reference
│   └── boards/                      # Microcontroller overlays
├── .github/workflows/               # CI/CD automation
│   └── build.yml                    # Firmware build pipeline
├── KEYMAP_GUIDELINES.md             # This file
├── CLAUDE.md                        # Claude Code integration
├── README.md                        # Project documentation
└── keymap.svg                       # Visual keymap
```

---

## Understanding Your Keyboard

### Hardware Layout

The Sofle Hybrid Ergomech has a symmetrical split layout:

```
Left Half:                              Right Half:
┌────┬────┬────┬────┬────┬────┐       ┌────┬────┬────┬────┬────┬────┐
│    │    │    │    │    │    │       │    │    │    │    │    │    │ Row 0
├────┼────┼────┼────┼────┼────┤       ├────┼────┼────┼────┼────┼────┤
│    │    │    │    │    │    │       │    │    │    │    │    │    │ Row 1
├────┼────┼────┼────┼────┼────┤       ├────┼────┼────┼────┼────┼────┤
│    │    │    │    │    │    │       │    │    │    │    │    │    │ Row 2
├────┼────┼────┼────┼────┼────┤       ├────┼────┼────┼────┼────┼────┤
│    │    │    │    │    │    │       │    │    │    │    │    │    │ Row 3
└────┴────┴────┴────┴────┴────┘       └────┴────┴────┴────┴────┴────┘
    ┌────┬────┬────┐                       ┌────┬────┬────┐
    │    │    │    │                       │ ↑  │    │    │ Thumb cluster
    ├────┼────┼────┤                       ├────┼────┼────┤ + 5-way switch
    │    │    │    │                       │ ↓  │    │    │
    └────┴────┴────┘                       └────┴────┴────┘
```

### Matrix Coordinates

When editing keymaps, keys are referenced by their position in the matrix. The format is: `ROW COL`

**Left half columns**: 0-5
**Right half columns**: 6-11 (offset by 6)

Example: `RC(0,0)` = Top-left key on left half, `RC(0,6)` = Top-left key on right half

---

## Keymap Basics

### What is a Layer?

A "layer" is like a keyboard profile or mode. You can have multiple layers and switch between them:

- **Layer 0 (Default)**: Your main typing layer
- **Layer 1, 2, 3**: Alternative layouts (gaming, coding, navigation, etc.)

You can switch between layers by:
- **Holding** a key (`&mo 1` = momentary toggle to layer 1)
- **Tapping** a key (`&lt 1 SPACE` = hold for layer 1, tap for space)
- **Toggling** permanently (`&tog 1` = toggle layer 1 on/off)

### Keymap File Format

```c
/ {
    keymap {
        compatible = "zmk,keymap";

        default_layer {
            label = "Default";
            bindings = <
                &kp ESC    &kp N1     &kp N2  ...  // Row 0
                &kp TAB    &kp Q      &kp W   ...  // Row 1
                &kp LCTRL  &kp A      &kp S   ...  // Row 2
                &kp LSHFT  &kp Z      &kp X   ...  // Row 3

                // Thumb cluster
                &kp LALT   &kp SPACE  &mo 1   ...  // Left thumb
                           ...                      // Right thumb
            >;
        };

        layer_name {
            label = "Layer Name";
            bindings = <
                // Different key layout
            >;
        };
    };
};
```

### Basic Key Syntax

```c
&kp KEY_CODE          // Press a key
&mo LAYER_NUM         // Momentary: hold to activate layer
&lt LAYER_NUM KEY     // Layer-tap: hold for layer, tap for key
&tog LAYER_NUM        // Toggle: switch layer on/off
&kp LSHIFT A          // Modifier + key (shift+A = capital A)
&kp LCTRL A           // Ctrl+A
```

---

## Creating a Custom Keymap

### Step 1: Open the Keymap File

Location: **`config/sofle_ergomech.keymap`**

Use your favorite text editor (VS Code, Sublime, Vim, etc.)

### Step 2: Understand the Current Layout

Review the default keymap to understand:
- Which keys are in which positions
- How layers are organized
- What special behaviors are used

### Step 3: Plan Your Layout

**Questions to ask yourself:**
- What is your primary use case? (typing, coding, gaming, etc.)
- Do you need multiple layers?
- Which keys do you use most frequently?
- Where do you want media controls?
- Do you need function keys readily accessible?

### Step 4: Modify Keys

Replace key codes with your preferences:

```c
// Change ESC to something else
- &kp ESC        // Original
+ &kp GRAVE      // Backtick instead

// Change a regular key to a layer toggle
- &kp DEL
+ &mo 1          // Hold to access layer 1

// Add a layer-tap (hold for layer, tap for key)
- &kp SPACE
+ &lt 1 SPACE    // Hold for layer 1, tap for space
```

### Step 5: Build and Test

1. Commit your changes: `git add . && git commit -m "custom keymap"`
2. Push to GitHub: `git push`
3. GitHub Actions automatically builds your firmware
4. Download and flash (see [Flashing Section](#flashing-your-keyboard))
5. Test your new keymap

### Step 6: Iterate

Don't get it perfect the first time! Keep refining your layout based on:
- Comfort and ergonomics
- Frequency of use for each key
- Muscle memory development
- Workflow optimization

---

## ZMK Key Codes Reference

### Letter & Number Keys

```c
A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z
N0, N1, N2, N3, N4, N5, N6, N7, N8, N9
```

### Modifier Keys

```c
LSHIFT, RSHIFT      // Left/Right Shift
LCTRL, RCTRL        // Left/Right Control
LALT, RALT          // Left/Right Alt
LGUI, RGUI          // Left/Right Windows/Command
```

### Navigation Keys

```c
UP, DOWN, LEFT, RIGHT
HOME, END
PGUP, PGDN          // Page Up/Down
```

### Special Keys

```c
SPACE               // Spacebar
ENTER               // Return/Enter
BSPC                // Backspace
DEL                 // Delete
TAB                 // Tab
ESC                 // Escape
GRAVE               // Backtick (~`)
MINUS               // Hyphen/Minus (-)
EQUAL               // Equal (+)
LBKT                // Left Bracket ([)
RBKT                // Right Bracket (])
BSLH                // Backslash (\)
SEMI                // Semicolon (;)
SQT                 // Single Quote (')
COMMA               // Comma (,)
DOT                 // Period (.)
FSLH                // Forward Slash (/)
```

### Function Keys

```c
F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, F11, F12
```

### Media Controls

```c
C_VOL_UP            // Volume Up
C_VOL_DN            // Volume Down
C_MUTE              // Mute
C_PLAY_PAUSE        // Play/Pause
C_NEXT              // Next Track
C_PREV              // Previous Track
C_BRI_UP            // Brightness Up
C_BRI_DN            // Brightness Down
```

### Media/System Keys

```c
C_PWR               // Power
C_SLEEP             // Sleep
C_WAKE              // Wake
C_AL_APP_MENU       // App Menu
C_AL_SEARCH         // Search
```

---

## Advanced Features

### Layer Switching

**Momentary Layer (Hold):**
```c
&mo 1               // Hold to activate layer 1 while held
```

**Layer Tap (Hold for Layer, Tap for Key):**
```c
&lt 1 SPACE         // Hold for layer 1, tap for spacebar
&lt 2 ENTER         // Hold for layer 2, tap for enter
```

**Toggle Layer (Press to Enable/Disable):**
```c
&tog 1              // Toggle layer 1 on/off with each press
```

### Modifier Combinations

Combine modifiers with keys:
```c
&kp LSHIFT A        // Shift+A (capital A)
&kp LCTRL C         // Ctrl+C
&kp LALT F4         // Alt+F4
&kp LGUI SPACE      // Windows/Cmd+Space
```

### Macros (Simple)

Define and use text macros:
```c
#define MAC(name, keys) \
    name: name { \
        compatible = "zmk,behavior-macro"; \
        label = #name; \
        #binding-cells = <0>; \
        bindings = <keys>; \
    };

behaviors {
    // Macro for common phrase
    hello_macro: hello_macro {
        compatible = "zmk,behavior-macro";
        label = "HELLO_MACRO";
        #binding-cells = <0>;
        bindings = <&kp H &kp E &kp L &kp L &kp O>;
    };
};

keymap {
    default_layer {
        bindings = <
            &hello_macro  // Tapping this key types "HELLO"
        >;
    };
};
```

### Combos (Key Combinations)

Trigger an action when multiple keys are pressed together:
```c
combos {
    compatible = "zmk,combos";

    combo_esc {
        timeout-ms = <50>;
        key-positions = <0 1>;  // Press first two keys together
        bindings = <&kp ESC>;
    };
};
```

### Encoder (5-Way Switch)

The right half has a 5-way directional switch configured as an encoder:
```c
// In the bindings for the thumb cluster right side
&kp UP              // Press encoder (up)
&kp DOWN            // Press encoder (down)
&kp LEFT            // Press encoder (left)
&kp RIGHT           // Press encoder (right)
&kp SPACE           // Press encoder (center/press)
```

---

## Flashing Your Keyboard

### Prerequisites

- USB-C cable
- Both keyboard halves
- Downloaded firmware files (`.uf2` format)
- Computer with USB port

### Build Your Firmware

#### Option 1: GitHub Actions (Recommended)

1. Go to your GitHub repository
2. Click **Actions** tab
3. Click **build** workflow
4. Click **Run workflow** (green button)
5. Wait for completion (2-3 minutes)
6. Click the completed run
7. Download artifacts:
   - `sofle_right-nice_nano_v2-zmk.uf2`
   - `sofle_left-nice_nano_v2-zmk.uf2`

#### Option 2: Local Build

Install ZMK and run:
```bash
west update
west build -b nice_nano_v2 -d build/right -- -DSHIELD=sofle_ergomech_right
west build -b nice_nano_v2 -d build/left -- -DSHIELD=sofle_ergomech_left
```

### Flash Right Half

1. **Connect** right half via USB-C
2. **Double-tap** the reset button on the Nice!Nano (back of PCB)
3. Keyboard appears as USB drive
4. **Drag and drop** `sofle_right-nice_nano_v2-zmk.uf2` onto the drive
5. **Wait** for flashing to complete
6. **Disconnect** USB

### Flash Left Half

1. **Connect** left half via USB-C
2. **Double-tap** reset button
3. Wait for USB drive to appear
4. **Drag and drop** `sofle_left-nice_nano_v2-zmk.uf2` onto the drive
5. **Wait** for completion
6. **Disconnect** USB

### Pair Keyboard Halves

1. **Power on** both halves
2. **Press pairing button** on each half (near reset button)
3. Go to system **Bluetooth settings**
4. Select and pair `sofle_ergomech` (appear as separate devices)
5. Both halves should now be connected

### Verify

- [ ] Both halves type correctly
- [ ] Function keys work
- [ ] Media controls respond
- [ ] OLED display shows status
- [ ] RGB lighting responds (if enabled)
- [ ] Bluetooth pairing is stable

---

## Troubleshooting

### Keyboard Not Recognized as USB Drive

**Problem**: Double-tapping reset doesn't work

**Solutions**:
1. Try double-tapping more quickly
2. Hold reset button for 2-3 seconds instead
3. Try a different USB cable (some cables are charge-only)
4. Check that the microcontroller is properly soldered
5. Try connecting different USB port

### Firmware Flashing Fails

**Problem**: File won't copy to keyboard

**Solutions**:
1. Verify you have the correct `.uf2` file (check right vs left)
2. Make sure the keyboard is actually in bootloader mode (should show USB drive)
3. Try using a different computer/USB port
4. The file must be named exactly as-is

### Only One Half Works

**Problem**: Only left or right half responds

**Solutions**:
1. Flash both halves (you need to flash left AND right separately)
2. Check Bluetooth pairing - pair both halves in settings
3. Ensure both halves are powered on
4. Try resetting both halves (hold reset button 5+ seconds)

### Bluetooth Connection Unstable

**Problem**: Keyboard keeps disconnecting

**Solutions**:
1. Bring the two halves closer together
2. Remove interference: move away from WiFi routers, Bluetooth speakers
3. Reduce the distance from your computer
4. Try re-pairing the keyboard
5. Check battery level on keyboard

### OLED Display Not Working

**Problem**: Display is blank or not showing status

**Solutions**:
1. Verify `CONFIG_ZMK_DISPLAY=y` in `config/sofle_ergomech.conf`
2. Ensure display is properly connected (solder joints)
3. Try rebuilding firmware with display enabled
4. Check display address in hardware config

### RGB LEDs Not Working

**Problem**: RGB underglow doesn't light up or respond

**Solutions**:
1. Verify `CONFIG_ZMK_RGB_UNDERGLOW=y` in config file
2. Check WS2812 LED strip connections
3. Ensure LEDs are getting power
4. Try resetting firmware with `settings_reset` file if available
5. Check LED order in hardware config

### Keys Are in Wrong Positions

**Problem**: Physical key presses produce wrong characters

**Solutions**:
1. Verify you're editing the correct layer
2. Check matrix coordinates - left is 0-5, right is 6-11
3. Make sure firmware was flashed to correct half (left/right)
4. Rebuild and reflash both halves
5. Compare with default keymap to find discrepancy

### Changes Not Taking Effect

**Problem**: Flashed firmware but keymap didn't update

**Solutions**:
1. Ensure you committed changes to GitHub
2. Wait for GitHub Actions build to complete
3. Verify you downloaded the latest firmware files
4. Check that you flashed BOTH halves
5. Try a full settings reset (double-tap reset + hold for 5 seconds)

### Recovery: Settings Reset

If everything is broken, reset the keyboard to factory defaults:

1. Download `settings_reset-nice_nano_v2-zmk.uf2` from artifacts
2. Flash to BOTH left and right halves:
   - Connect left, double-tap reset, drag file, wait
   - Connect right, double-tap reset, drag file, wait
3. This clears all Bluetooth pairings and resets to defaults
4. Re-pair and re-flash your keymap

---

## Tips & Best Practices

### Ergonomics

- Place most-used keys on thumbs and home row
- Avoid reaching for uncommon keys on corners
- Use layer switches for less frequent keys
- Test layout for at least 1 week before making major changes

### Optimization

- Learn your layers - consistency reduces muscle memory confusion
- Keep layers logically organized (layer 1 = alternate, layer 2 = functions, etc.)
- Group related keys together when possible
- Leave some keys empty initially, add as needed

### Backups

- Keep your keymap in version control (Git)
- Document your layer organization in comments
- Save screenshots of your layout
- Keep a reference of what each layer does

### Testing

- Flash one change at a time, not multiple
- Use the visual keymap tool to verify placement
- Test in actual use before major changes
- Have a backup default layout to revert to

---

## Additional Resources

- [ZMK Documentation](https://zmk.dev)
- [ZMK Key Codes](https://zmk.dev/docs/codes/keyboard-keymap)
- [ZMK Behaviors](https://zmk.dev/docs/behaviors/overview)
- [Nice!Nano Documentation](https://nicekeyboards.com/#/)
- [Sofle Keyboard Original](https://github.com/soflettkydid/Sofle)

---

## Getting Help

If you encounter issues:

1. Check this guide's [Troubleshooting](#troubleshooting) section
2. Review the [ZMK Documentation](https://zmk.dev)
3. Check GitHub issues in this repository
4. Ask in ZMK Discord community

---

**Last Updated**: 2025-11-13
**Keyboard**: Sofle Hybrid Ergomech Rev 5
**Firmware**: ZMK
