# Keebart Miryoku ZMK Firmware

This repository builds [Miryoku](https://github.com/manna-harbour/miryoku) firmware for the following Keebart Bluetooth keyboards:

| Keyboard | GitHub Actions workflow | Boards built |
| --- | --- | --- |
| Corne Choc Pro BT | `Build Corne Choc Pro BT` | `corne_choc_pro_left`, `corne_choc_pro_right` |
| Piantor Pro BT (42-key) | `Build Piantor Pro BT` | `piantor_pro_bt_left`, `piantor_pro_bt_right` |
| Piantor Pro BT (36-key, 5-col) | `Build Piantor Pro BT 5-Col` | `piantor_pro_bt_5col_left`, `piantor_pro_bt_5col_right` |
| Sofle Choc Pro BT | `Build Sofle Choc Pro BT` | `sofle_choc_pro_left`, `sofle_choc_pro_right` |

Each workflow builds both halves and includes the Keebart Sharp MIP display shield. ZMK Studio is disabled in all builds from this repository.

The firmware is based on [Miryoku ZMK](https://github.com/manna-harbour/miryoku_zmk) and [ZMK Firmware](https://zmk.dev/).

## Build firmware with GitHub Actions

GitHub Actions is the recommended build method. It does not require a local ZMK development environment.

### First-time setup

1. Sign in to GitHub and fork this repository.
2. Open the fork and select the **Actions** tab.
3. If GitHub asks for confirmation, select **I understand my workflows, go ahead and enable them**.

### Start a build

1. Open the **Actions** tab in your fork.
2. Select the workflow for your keyboard:
   - `Build Corne Choc Pro BT` for Corne Choc Pro BT.
   - `Build Piantor Pro BT` for Piantor Pro BT (42-key, with thumb cluster).
   - `Build Piantor Pro BT 5-Col` for Piantor Pro BT (36-key: 5 columns per hand instead of 6, still with the 3-key thumb cluster per hand).
   - `Build Sofle Choc Pro BT` for Sofle Choc Pro BT.
3. Select **Run workflow**.
4. Select the branch to build.
5. Choose the desired **Alpha layout**.
6. Select **Run workflow** and wait for both build jobs to finish.

The available alpha layouts are:

- AZERTY
- BEAKL15
- Colemak
- Colemak-DHk (`ColemakDHK`)
- Dvorak
- Halmak
- QWERTY
- QWERTZ
- Workman

QWERTY is selected by default. The selected layout applies to both halves.

### Download the firmware

1. Open the completed workflow run.
2. Scroll to **Artifacts** at the bottom of the summary page.
3. Download the artifacts for the left and right boards.
4. Unzip both downloads and identify the firmware file for each side.

Artifact names include the keyboard side and selected settings. Do not flash the left firmware onto the right half or the right firmware onto the left half.

### Flash the keyboard

Repeat these steps for each half:

1. Connect the half to the computer over USB.
2. Put it into bootloader mode by quickly pressing reset twice.
3. Copy the downloaded `.uf2` firmware file for that side to the USB drive that appears.
4. Wait for the keyboard to reboot and the USB drive to disappear.

## Build with different Miryoku settings

The three Keebart workflows expose the alpha layout as a dropdown. For other Miryoku settings, edit the corresponding workflow in your fork under [`.github/workflows`](.github/workflows) and add options below its existing `with:` block.

Workflow values are JSON arrays. A single value has this form:

```yaml
option: '["value"]'
```

For example, the following uses Colemak-DHk alphas, vi-style navigation, flipped layers, and Windows clipboard shortcuts:

```yaml
with:
  board: '["corne_choc_pro_left","corne_choc_pro_right"]'
  extra_shield: '["sharp_mip"]'
  alphas: '["ColemakDHK"]'
  nav: '["vi"]'
  layers: '["flip"]'
  clipboard: '["win"]'
```

Replacing the workflow's dynamic `alphas` line with a fixed value, as in this example, removes the effect of the alpha-layout dropdown. Keep the existing line below if the dropdown should continue to control alphas:

```yaml
alphas: ${{ format('["{0}"]', inputs.alphas) }}
```

Supported common options include:

| Option | Values | Default behaviour |
| --- | --- | --- |
| `alphas` | `AZERTY`, `BEAKL15`, `Colemak`, `ColemakDHK`, `Dvorak`, `Halmak`, `QWERTY`, `QWERTZ`, `Workman` | Colemak-DH |
| `extra` | The same layout values as `alphas` | QWERTY |
| `tap` | The same layout values as `alphas` | Colemak-DH |
| `nav` | `invertedT`, `vi` | Miryoku home-row navigation |
| `clipboard` | `fun`, `mac`, `win` | CUA bindings |
| `layers` | `flip` | Navigation on the right hand |

Use `["default"]` or omit an option to keep its Miryoku default.

Multiple values can be supplied to create a build matrix. For example:

```yaml
alphas: '["QWERTY","ColemakDHK"]'
nav: '["default","vi"]'
```

This builds every combination of the supplied values for both keyboard halves. Large matrices can produce many jobs and artifacts.

For Miryoku settings not represented by a workflow option, use `custom_config` with the corresponding preprocessor definition:

```yaml
custom_config: '["#define MIRYOKU_CLIPBOARD_WIN"]'
```

See the upstream [Miryoku reference](https://github.com/manna-harbour/miryoku/tree/master/docs/reference) for available layout and layer settings.

The reusable workflow also accepts a `kconfig` option for advanced ZMK configuration. `CONFIG_ZMK_STUDIO=n` is always applied after custom Kconfig, so ZMK Studio cannot be enabled by a Keebart Miryoku build.

## Use a custom Keebart `zmk-config` fork

The keyboard board definitions and Sharp MIP display shield come from [Keebart/zmk-config](https://github.com/Keebart/zmk-config), which this repository loads as an external ZMK module. Use your own fork when you have changed the display shield, a board definition, a board-level Kconfig default, or another part of the Keebart hardware module.

### Select your fork

1. Fork [Keebart/zmk-config](https://github.com/Keebart/zmk-config).
2. Make and commit your changes on a branch in that fork.
3. In your fork of this Miryoku repository, open the outboard file for the keyboard you want to build:
   - [Corne Choc Pro BT outboard](.github/workflows/outboards/boards/corne_choc_pro)
   - [Piantor Pro BT outboard](.github/workflows/outboards/boards/piantor_pro_bt)
   - [Piantor Pro BT 5-Col outboard](.github/workflows/outboards/boards/piantor_pro_bt_5col)
   - [Sofle Choc Pro BT outboard](.github/workflows/outboards/boards/sofle_choc_pro)
4. Replace the existing module reference:

   ```sh
   outboard_modules=Keebart/zmk-config/main
   ```

   with your GitHub account, repository, and branch:

   ```sh
   outboard_modules=YOUR_GITHUB_USERNAME/zmk-config/YOUR_BRANCH
   ```

5. Commit the outboard change and run the normal keyboard workflow from the **Actions** tab.

The build clones the selected branch and passes it to ZMK as an extra module. Keep the module structure from the Keebart repository, including `zephyr/module.yml` and the board and shield directories. The fork and branch must be readable by the workflow; a public fork works with the current configuration.

Replace the existing outboard reference rather than adding your fork through the workflow's `modules` option. Otherwise the official Keebart module and your fork are loaded together and may provide conflicting definitions for the same boards or shield.

If you want all the Keebart keyboard workflows to use your fork, update all of the outboard files listed above. You can point each keyboard at a different fork or branch when testing changes independently.

### What comes from each repository

- Your `zmk-config` fork supplies the Corne Choc Pro BT, Piantor Pro BT, and Sofle Choc Pro BT board definitions and the `sharp_mip` display shield.
- This Miryoku repository supplies the keyboard `.keymap` files and Miryoku settings.
- Keymap files under your `zmk-config` fork's `config/` directory do not replace the Miryoku keymaps used here. Change the Miryoku workflow settings, `custom_config`, or the files in this repository's [`config`](config) directory when you want to change the compiled Miryoku keymap.

### Use your own ZMK base firmware

A `zmk-config` fork is a hardware module, not the ZMK base source. If your changes are instead based on your own fork of `zmkfirmware/zmk`, add the `branches` option to the Keebart keyboard workflow's `with:` block:

```yaml
branches: '["YOUR_GITHUB_USERNAME/zmk/YOUR_BRANCH"]'
```

The first `user/repository/branch` value selects the ZMK source that the workflow clones. This can be combined with the custom `zmk-config` outboard reference described above: `branches` selects your ZMK base firmware, while `outboard_modules` selects your Keebart hardware module.

The Miryoku workflow still forces `CONFIG_ZMK_STUDIO=n`, including when a custom ZMK base or custom Keebart module would otherwise enable Studio.

## Repository layout

- [Corne Choc Pro BT workflow](.github/workflows/build-corne-choc-pro.yml)
- [Piantor Pro BT workflow](.github/workflows/build-piantor-pro.yml)
- [Piantor Pro BT 5-Col workflow](.github/workflows/build-piantor-pro-5col.yml)
- [Sofle Choc Pro BT workflow](.github/workflows/build-sofle-choc-pro.yml)
- [Shared build workflow](.github/workflows/main.yml)
- [Corne Choc Pro BT keymap](config/corne_choc_pro.keymap)
- [Piantor Pro BT keymap](config/piantor_pro_bt.keymap)
- [Piantor Pro BT 5-Col keymap](config/piantor_pro_bt_5col.keymap)
- [Sofle Choc Pro BT keymap](config/sofle_choc_pro.keymap)
- [Shared Miryoku configuration](miryoku/custom_config.h)

## Credits

- [Keebart](https://keebart.com/) keyboard hardware and board definitions
- [Miryoku](https://github.com/manna-harbour/miryoku) by Manna Harbour
- [ZMK Firmware](https://zmk.dev/)
