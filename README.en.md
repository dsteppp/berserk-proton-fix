# Berserk Heroes Online: Steam version sign-in fix for Linux

[Русский](README.md)

Under Proton, the game may start but show an error instead of the email sign-in window. This script fixes that window in the **Steam version of Berserk Heroes Online** on Linux x86_64 and Steam Deck.

## Why sign-in failed

The game opens its sign-in page using Microsoft WebView2. Under Proton, this window could fail to appear or remain blank. The plugin uses DirectComposition, a rendering path that returned `0x80004001` ("not implemented") under Proton. WebView2 and Steam registry paths could also be missing.

The patch replaces the open-source `WebView.dll` plugin. Sign-in is embedded in the game window and uses software rendering. On Steam Deck, this also fixes on-screen keyboard input in Gaming Mode. The game's own graphics are unchanged.

A clean install and sign-in were tested on a Linux PC. Updating an existing patch and using the on-screen keyboard were also tested on Steam Deck in Gaming Mode.

**This is a temporary, unofficial workaround until the developers provide full Linux support.** It does not imply that such support has been announced. Compatibility with every device is not guaranteed.

## Install

> **Steam Deck: install the patch in Desktop Mode. Once installation finishes, return to Gaming Mode and launch the game from your Steam library.** Sign-in and the on-screen keyboard were tested in Gaming Mode.

You need Steam with an account signed in, **the game already installed through Steam**, and an internet connection. You do not need to launch the game first. Any Steam library can be used, including another drive or a microSD card.

Allow at least 3 GB for the game environment, plus space for a backup if an environment already exists. Proton and the Steam runtime need additional download space.

1. Close running games. On Steam Deck, switch to **Desktop Mode**.
2. Download [berserk-proton-fix.sh](berserk-proton-fix.sh). Open a terminal in its folder and run:

   ```bash
   bash berserk-proton-fix.sh
   ```

3. Confirm tool installation if Steam asks. Once the script finishes, launch the game through Steam and sign in by email. **On Steam Deck, return to Gaming Mode first, then launch the game from your Steam library.**

The script selects Proton and sets launch options automatically. To do this, it requests a normal Steam shutdown, then reopens Steam afterwards. Do not reopen Steam yourself during installation.

Bash, Python 3, curl and standard Linux commands are required. If something is missing or settings cannot be changed safely, the script stops with instructions. It does not use sudo, install system packages or disable SteamOS protection.

## What the script does

- Finds the game by AppID `4926310`, without a fixed installation path.
- If needed, opens Steam's installation of official Proton 11.0 and the required Steam Linux Runtime.
- Saves the original state for rollback.
- Downloads WebView2 from Microsoft and installs it only in this game's environment.
- Adds Steam paths and WebView2 settings to the environment's registry.
- Replaces `WebView.dll` and adds `PROTON_USE_XALIA=0` to launch options, preserving ordinary existing options.

The helper and DLL are embedded in the single `.sh` file. No separate build or download is needed for them.

## Why there is a backup

**The script creates the backup before its first changes. You do not need to prepare one yourself.** It saves the original DLL, affected Steam settings and the prefix: Proton's folder containing Windows components, settings and sign-in data for this game. If no prefix exists yet, the script records that instead.

The backup is only for returning to the state before patch installation. It is not a backup of the entire game or Steam. Running the installer again does not replace it or erase a saved sign-in.

The backup, WebView2 installer and logs are kept in **one folder beside the prefix**: `steamapps/compatdata/.berserk-proton-fix-4926310`. The script creates no separate folders in your home directory. Backup size depends on the existing prefix. **Do not publish this folder: it may contain authentication data.**

## Updating the patch

If the patch is already installed, download the new `berserk-proton-fix.sh` and run the same command. **No rollback is needed first.** The script updates the DLL, preserves your saved sign-in and keeps the original rollback backup. Components already installed are not downloaded or installed again.

## If installation stops or the game updates

Follow the script's instructions, then run the same installation command again. Completed components are reused. If Steam is still downloading tools, let it finish before retrying.

If sign-in stops working after a game update, **try reinstalling the patch with the same command**. Steam may have restored the original DLL. However, the developer may have changed the plugin or sign-in flow, making the old patch incompatible. **Compatibility after updates is not guaranteed.** If the script reports an incompatible DLL or the problem persists, do not bypass its checks or manually restore an old DLL.

Check the environment without installing anything:

```bash
bash berserk-proton-fix.sh --check
```

This checks components and access to settings, not account sign-in.

## Full rollback

Close the game and run:

```bash
bash berserk-proton-fix.sh --uninstall
```

The script restores the backed-up DLL, prefix and settings. If no prefix existed before installation, it removes the created one. It then deletes the backup, logs and its cache. You can delete the downloaded `.sh` file yourself.

**Sign-in and settings created after the first patch installation will be lost.** If the DLL was updated or you changed launch options, rollback may stop rather than overwrite newer data. Follow its instructions.

Other games and their settings are unchanged. Proton and Steam Linux Runtime remain because other games may need them. The script does not undo manual changes made before its installation.

## Rights and terms

The patch changes only the open-source sign-in plugin, the local environment and Steam settings. It does not change gameplay rules, assets, the main executable or server authentication. It does not bypass payments, DRM, anti-cheat or access checks. It is not affiliated with the game developer, Valve or Microsoft.

The plugin is a modified build of [gree/unity-webview](https://github.com/gree/unity-webview), revision `1256c3849c`, under the [zlib license](LICENSE.unity-webview). Original code and modifications are [MIT-licensed](LICENSE). The game and third-party installers are not redistributed here. Their owners' rights and terms still apply.

**Provided as is, without guarantees or commitments to support or review PRs.** For changes, fork the project and maintain your own version. Read third-party scripts before running them.
