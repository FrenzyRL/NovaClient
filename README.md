# NovaLauncher

NovaLauncher is a small desktop launcher for **Minecraft: Java Edition** that installs
[Fabric](https://fabricmc.net/) and the **NovaClient** mod and starts the game.
It is written in Python with a [pywebview](https://pywebview.flowrl.com/) (HTML/CSS/JS) interface.

> NovaLauncher is an unofficial third-party project. It is not affiliated with, endorsed by or
> supported by Mojang Studios or Microsoft. It does not contain or redistribute any Minecraft game files:
> everything is downloaded from the official Mojang / Fabric servers on the player's own computer.

## Online mode only - real accounts only

- Players sign in with their **own Microsoft account**. There is **no offline mode** and no way to enter a
  username without signing in.
- The game only starts with an account that **owns Minecraft: Java Edition**. If the account does not own the
  game, the launcher shows an error and does not start it.
- The launcher does not bypass or disable authentication, ownership/licence checks or any other safety feature.

### How sign-in works

1. The launcher opens the Microsoft sign-in page (OAuth 2.0 authorization code flow with **PKCE** and a `state`
   check). The password is typed on Microsoft's own page and never reaches the launcher.
2. Scope requested: `XboxLive.signin offline_access` - nothing else (no mail, files or other Microsoft data).
3. The authorization code is exchanged for a Microsoft token, then for Xbox Live (XBL), XSTS and finally
   **Minecraft Services** (`login_with_xbox`) tokens. The Minecraft profile is read from the Minecraft Services
   profile endpoint.
4. On every launch the stored refresh token is used to get a fresh access token. If the refresh token is
   rejected, the player is asked to sign in again.

The app is a **public client** (no client secret) and has **no backend server**. All requests go directly from
the player's computer to Microsoft, Xbox Live and Minecraft Services.

### Data stored locally

Stored on the player's own computer only, in `%APPDATA%\NovaLauncher\` (Windows) or the platform equivalent:

| File | Content |
| --- | --- |
| `accounts.json` | account name, UUID and the Microsoft **refresh token** (treated as a secret, never shown in the UI) |
| `config.json` | launcher settings (selected version, RAM, Java path, ...) |
| `instances/<version>/` | game folder (mods, saves, options) |
| `minecraft/` | shared game files (libraries, assets, Java runtime) |
| `game_output.log` | output of the last game session |

Signing out in the launcher deletes the stored token. The refresh token is currently stored as plain text in
`accounts.json` (file permissions are restricted on Linux/macOS); do not share this file. The permission can also
be revoked at any time at <https://account.live.com/consent/Manage>.

No telemetry, no analytics, no advertising.

## Features

- Microsoft sign-in with multiple accounts
- Fabric + Fabric API + NovaClient installed automatically, one game folder per Minecraft version
- Mod manager: enable / disable / delete mods, add your own `.jar`, search and install from [Modrinth](https://modrinth.com/)
- Optional automatic server join on start, RAM / Java / JVM argument settings, live game log

## Requirements

- Windows 10/11 (WebView2 runtime; included in Windows 11), Python 3.10+
- A Microsoft account that owns Minecraft: Java Edition
- Internet connection for the first start of each Minecraft version

## Run from source

```bash
pip install -r requirements.txt
python main.py          # add --debug for the web inspector
```

On Windows you can simply run `run.bat`. `build_exe.bat` builds a single `NovaLauncher.exe` with PyInstaller.

## Using your own Azure app

Sign-in needs an Azure app registration (public client). To build your own copy:

1. Register an app in the Azure portal (**Personal Microsoft accounts only**), platform **Mobile and desktop
   applications**, redirect URI `https://login.microsoftonline.com/common/oauth2/nativeclient`, and enable
   **Allow public client flows**.
2. Put the Application (client) ID in `core.py` (`AZURE_CLIENT_ID`), in the launcher's Settings, or in the
   `NOVA_CLIENT_ID` environment variable.
3. New app IDs must be approved by Mojang before Minecraft Services accepts them
   (<https://aka.ms/mce-reviewappid>). Until then sign-in stops with an "app not approved" message.
4. In `core.py` set `GITHUB_REPO` to the repository whose releases contain the NovaClient jars.

NovaClient jars are published as GitHub release assets and must contain the Minecraft version in the file name,
for example `novaclient-mc1.21.4-1.0.0.jar`.

## Project layout

```
main.py      entry point (pywebview window)
api.py       Python API exposed to the web UI
core.py      authentication, installation, mod manager
web/         HTML / CSS / JS interface
assets/      fonts
```

## Licence and credits

- Authentication and game installation use [minecraft-launcher-lib](https://codeberg.org/JakobDev/minecraft-launcher-lib).
- Font: Poppins (SIL Open Font License, see `assets/fonts/LICENSE-Poppins.txt`).
- "Minecraft" is a trademark of Mojang Studios. All rights to the game belong to their respective owners.

Add your own licence file (for example MIT) and replace this section with it.
