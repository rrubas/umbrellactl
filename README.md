# umbrellactl

Utility script for macOS that checks the status of the Cisco Umbrella Roaming Security module, and can enable or disable it on Cisco Secure Client (AnyConnect) deployments — including the 5.1.x plugin layout.

The script detects whether the Umbrella libraries live in the legacy location (`/opt/cisco/secureclient/bin/plugins`) or under the Secure Client 5.1 umbrella bundle (`/opt/cisco/secureclient/bin/umbrella/plugins`) and acts accordingly.

Official guidance on manually disabling or restarting the Umbrella client is available from Cisco: https://support.umbrella.com/hc/en-us/articles/230561067-Umbrella-Roaming-Client-Manually-Disabling-or-Restarting

## Requirements

- macOS with Cisco Secure Client / AnyConnect installed.
- Administrator privileges — enabling or disabling the module requires moving system files, so you will be asked for your password the first time `sudo` runs.

## Quick start options

Pick the variant that fits your situation. All commands below can be pasted exactly as shown.

### Option 1 — Run once without downloading

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/rrubas/umbrellactl/main/umbrellactl) -s
```

- Copy and paste the command above into Terminal and press Enter.
- Add `-e` or `-d` at the end if you want to enable or disable the module instead of checking status.
- Omit the final parameter to open the interactive menu. Example: `bash <(curl -fsSL https://raw.githubusercontent.com/rrubas/umbrellactl/main/umbrellactl)`
- Requires an internet connection and trusts the latest version in the repository.

### Option 2 — Use a local copy (clone or download)

Starting from scratch?

```bash
git clone https://github.com/rrubas/umbrellactl.git
cd umbrellactl
chmod +x umbrellactl          # one-time setup
./umbrellactl -s              # check current status
```

Already have this folder?

```bash
cd umbrellactl
git pull
chmod +x umbrellactl          # refresh permissions if needed
./umbrellactl -s
```

Use `./umbrellactl -e` or `./umbrellactl -d` to enable or disable the module. `sudo` will prompt for your password when the script needs elevated permissions.

- Works without an internet connection after the initial download.
- Lets you inspect the script before running it.

### Option 3 — Install system-wide (optional)

Install from a local checkout:

```bash
sudo install -m 755 umbrellactl /usr/local/bin/umbrellactl
```

Install directly from GitHub (no checkout needed):

```bash
sudo sh -c 'curl -fsSL https://raw.githubusercontent.com/rrubas/umbrellactl/main/umbrellactl \
  -o /usr/local/bin/umbrellactl && chmod 755 /usr/local/bin/umbrellactl'
```

After installation you can run the tool as `umbrellactl -s` (or `-e` / `-d`) from any terminal. Removing it is as simple as deleting `/usr/local/bin/umbrellactl`.

- Ideal if you manage multiple machines and want a consistent command.
- Keep in mind that updates require re-running the same install command.

## Usage

```
umbrellactl [-s|-e|-d|-h]

    -s, --status    Print Umbrella Roaming Security module status.
    -e, --enable    Enable the module (moves the dylibs back into the active plugin directory).
    -d, --disable   Disable the module (moves the dylibs into the disabled/ directory).
    -h, --help      Show this message.

Running `umbrellactl` without arguments (or with an unknown option) opens an interactive menu where you can choose to check the status, enable/disable the module, or exit; the menu returns after each action until you choose to exit.
```

- `-s` works without privileges; the script only reads the filesystem.
- `-d` and `-e` require administrator privileges because they relocate the Umbrella plugin libraries. Run the commands and supply your password when `sudo` prompts.
- Success messages appear in green, while errors are highlighted in red to make them easy to spot.

## How it works

1. Detects the active Umbrella plugin directory (legacy vs. Secure Client 5.1 umbrella bundle).
2. Confirms whether the three Umbrella dylibs (`libacumbrellaapi.dylib`, `libacumbrellactrl.dylib`, `libacumbrellaplugin.dylib`) are active or already under `disabled/`.
3. Moves those libraries as requested, mirroring Cisco’s manual disable/enable guidance for 5.1.x clients.

If the script cannot determine a consistent state (for example, files are missing from both locations), it prints an error so you can investigate before taking further action.
