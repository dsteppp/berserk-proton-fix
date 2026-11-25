# Berserk Heroes Online: 4game sign-in on Linux

[Русский](README.md)

An unofficial compatibility patch for the **Steam version of Berserk Heroes Online** (AppID `4926310`). It helps display the email sign-in window under Proton. The same installer is intended for Linux x86_64 PCs and Steam Deck.

**This is a temporary workaround until the game developers provide full official Linux support.** It does not replace official support or imply that the developers have committed to releasing it.

## Before you start

- Install Steam and sign in.
- **Install the game through Steam.** Any Steam library is supported in principle, including another drive or a microSD card.
- Have an internet connection for missing components.
- Allow up to 3 GB for the game environment, plus space for one backup of an existing Proton prefix. Steam needs additional space if it must download Proton and its runtime.

**You do not need to launch the game first.** If you already launched it with another Proton version, the installer backs up the existing environment and selects Proton itself. Close the game before running the installer.

Bash, Python 3, curl and standard Linux commands are required. The script checks for them. It does not use sudo, install system packages or disable SteamOS read-only protection. Having Steam installed does not guarantee these commands exist on every Linux distribution.

## Install

1. On Steam Deck, switch to **Desktop Mode**. On a Linux PC, open a terminal.
2. Download `berserk-proton-fix.sh` from this repository. Open a terminal in the folder containing the downloaded file.
3. Run:

   ```bash
   bash berserk-proton-fix.sh
   ```

4. If Steam displays a Proton or Steam Linux Runtime installation dialog, confirm it. The script handles the remaining steps. If a download takes longer than the wait period, let Steam finish and run the same command again.
5. When installation completes, launch the game **through Steam** and sign in by email. On Deck you can then return to Gaming Mode.

You do not need to choose Proton or edit launch options manually. The script selects Proton and adds `PROTON_USE_XALIA=0`, preserving ordinary existing options. It requests a normal Steam shutdown before editing settings and reopens Steam afterwards. Do not reopen Steam yourself while installation is running.

If existing launch options contain a complex command the installer cannot safely adjust, it stops and explains how to save those options and temporarily clear the field. Do not discard your settings without a copy.

### If installation stops

Read the error message, follow its instructions, then run:

```bash
bash berserk-proton-fix.sh
```

Completed components are reused. An interrupted WebView2 download resumes using its saved Microsoft download URL. The first completed backup is not replaced. A normal retry does not erase a saved sign-in.

**If sign-in stops working after a game update, try running the installer again:**

```bash
bash berserk-proton-fix.sh
```

Steam may have replaced the patched DLL with the original; reinstalling the patch may help in that case. **Compatibility after updates is not guaranteed:** the developer may change the plugin, its interface with the game or the authentication flow. The existing patch may then be incompatible, and reinstalling it cannot fix that incompatibility. If the problem persists, stop using the patch with that game version. Steam file verification may also restore the original DLL.

The full-rollback snapshot belongs to the game version present at the first installation. After an update, do not manually restore an old DLL: it may be incompatible with the new game version.

Check the environment **without installing anything or changing settings**:

```bash
bash berserk-proton-fix.sh --check
```

This checks paths, tools and whether settings can be prepared. It does not test account sign-in or prove the game works.

## Full rollback

Close the game and run from the folder containing the script:

```bash
bash berserk-proton-fix.sh --uninstall
```

This restores:

- the DLL present before the first installation using this installer;
- the original Proton prefix, or removes the created prefix if none existed before;
- the previous Proton selection and launch options for this game, without reverting playtime or other games' settings.

It then deletes this patch's backup, logs and download cache. You can delete the downloaded `.sh` file yourself.

**Sign-in and settings created after the first installation are lost during full rollback.** The backup may contain authentication data; do not publish it. If you subsequently changed launch options or Steam updated the DLL, rollback may stop rather than overwrite newer data. Follow the message instead of deleting files at random.

Proton and Steam Linux Runtime remain installed because other games may use them. The script does not remove Steam or the game. Manual modifications and leftovers from older installation methods predating this installer are not automatically undone.

## Why sign-in failed

The game embeds the 4game sign-in page using Microsoft WebView2. Its original open-source Unity plugin uses `CreateCoreWebView2CompositionController`. Under the tested Proton version, this returned `E_NOTIMPL` (`0x80004001`): the required DirectComposition path did not work. A clean prefix could also lack WebView2 or Steam registry paths.

The modified plugin uses `CreateCoreWebView2Controller`, displaying a regular browser window over the game. WebView2 receives software-rendering options: `--disable-gpu --disable-gpu-compositing`. These affect the sign-in window, not the game's rendering. There are no synthetic activation clicks or sandbox-disabling flags. `PROTON_USE_XALIA=0` disables Xalia for this launch because its activity interfered with WebView2 in testing.

## What the installer does

1. Finds Steam, its libraries and the game by AppID manifests, not a hard-coded installation folder.
2. Selects official Proton 11.0. Reads the required Steam Linux Runtime from Proton's manifest. Incomplete downloads are not treated as installed tools.
3. Opens Steam's normal tool installation if required. It does not download third-party Proton builds.
4. Requests normal Steam shutdown and saves the original DLL, prefix and affected settings. The backup is stored in `steamapps/compatdata/.berserk-proton-fix-4926310`, beside the prefix.
5. Sets the selected Proton and launch option for local Steam users, without changing other games' settings.
6. If necessary, runs Microsoft's official WebView2 installer inside this game's prefix through the matching Steam runtime. The installer is downloaded over HTTPS from Microsoft, not bundled here.
7. Adds Steam paths and WebView2 compatibility settings to the prefix registry and replaces `WebView.dll` with the included modified open-source plugin. The embedded DLL checksum is verified before replacement.
8. Reuses completed steps and reopens Steam afterwards. Errors include a reason, log location and rollback command.

The cache is stored under `$XDG_CACHE_HOME/berserk-proton-fix` or `~/.cache/berserk-proton-fix`. For one environment, storage includes one WebView2 installer up to 512 MiB, up to two Proton logs capped at 4 MiB each, and one prefix snapshot. Snapshot size depends on the existing prefix; this is not a general Steam backup. Do not attach the `compatdata` directory to public bug reports.

## Purpose, rights and licenses

This is an independent, unofficial compatibility modification, not affiliated with the game developer, 4game, Valve or Microsoft. It does not add gameplay features, change balance, assets, the main game executable or server authentication. It does not bypass access checks, payments, DRM or anti-cheat.

It would nevertheless be inaccurate to say that no game files are changed: the script replaces the **open-source `WebView.dll` plugin** and modifies the local prefix and Steam settings. The original DLL is saved for rollback. This repository does not redistribute the game, its assets, Steam, Proton or the WebView2 installer.

The plugin is based on [gree/unity-webview](https://github.com/gree/unity-webview), `plugins/Windows`, revision `1256c3849c`. This is a modified build, not the original upstream plugin. Its **zlib license** permits modification and redistribution subject to preserving attribution and marking altered versions. The original notice is preserved in `LICENSE.unity-webview`. The installer and original modifications are MIT-licensed (`LICENSE`). Rights to the game and third-party components remain with their respective owners. Their terms still apply; no blanket legal assurance concerning those terms is made here.

## Use and independent development

This project is provided **as is**. There are no guarantees, support commitments, promises to review third-party builds or obligations to accept contributions. It is a published workaround, not a maintained service. Use it if it helps. If you want changes, fork it and maintain your version independently. Responses to issues and pull-request reviews are not promised.

Only the single `.sh` file is needed: the helper and patched DLL are embedded. No separate build is required. For independent changes, inspect the script and consult the upstream unity-webview project. Review third-party modified scripts before running them.
