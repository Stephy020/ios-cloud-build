# ios-cloud-build

Build your Expo / React Native app for iPhone **without a Mac and without the paid Apple Developer Program**, from Linux or Windows (Git Bash).

```
your PC                         GitHub (free Mac)                    your PC → iPhone
ios-cloud run ──git push──▶  builds an UNSIGNED .ipa  ──download──▶  signs it with your free
                             (Xcode, CocoaPods, no                   Apple ID and installs it
                              Apple account needed)                  (xtool / Impactor)
```

- The GitHub Mac never signs anything, so it needs no Apple account or certificates.
- Your PC signs the app with your **free Apple ID** and installs it over USB.
- It works for any Expo project, and for bare React Native projects that have an `ios/` folder.

## One-time setup

### Linux

1. Install [Node.js](https://nodejs.org) 22+, git, and the [GitHub CLI](https://github.com/cli/cli/releases) (`gh`).
2. Install [xtool](https://xtool.sh) and log in with your Apple ID: `xtool setup`
3. Install this toolkit:
   ```bash
   git clone https://github.com/Stephy020/ios-cloud-build ~/ios-cloud-build
   ~/ios-cloud-build/install.sh
   ```
4. Log in to GitHub: `gh auth login -s workflow`

### Windows (Git Bash, no WSL)

1. Install [Git for Windows](https://git-scm.com/download/win), which includes Git Bash. Then in PowerShell or Git Bash:
   ```bash
   winget install OpenJS.NodeJS.LTS
   winget install --id GitHub.cli
   ```
2. Install **iTunes** from apple.com. It provides the iPhone USB driver.
3. Download **plumesign** (the command-line tool of [Impactor](https://github.com/claration/Impactor/releases)) for your CPU, for example `plumesign-windows-x86_64.exe`. Save it as `C:\Users\<you>\bin\plumesign.exe`.
4. Open a **new** Git Bash window and run:
   ```bash
   git clone https://github.com/Stephy020/ios-cloud-build ~/ios-cloud-build
   ~/ios-cloud-build/install.sh
   gh auth login -s workflow
   plumesign account login -u you@example.com     # your Apple ID
   ```

Prefer the Impactor app (GUI)? Skip plumesign. `ios-cloud install` will then open the folder with the `.ipa`, and you drag it into Impactor. If you already use xtool inside WSL, ios-cloud can use that too (`IOS_CLOUD_INSTALLER=wsl`).

### On the iPhone (once)

- **Settings → Privacy & Security → Developer Mode → On** (the iPhone restarts).
- After the first install: **Settings → General → VPN & Device Management →** your Apple ID **→ Trust**.

Check everything with:

```bash
ios-cloud doctor
```

## Use it in a project

```bash
cd my-app
ios-cloud init                # once per project: adds the workflow, expo-dev-client, creates the GitHub repo
ios-cloud run release         # build in the cloud, download, install and launch on your iPhone
```

What `init` does:

- Adds `.github/workflows/ios-build.yml`.
- Makes sure `ios.bundleIdentifier` is set.
- Adds `expo-dev-client`.
- Creates a **public** GitHub repo and pushes. Use `--private` for a private repo, or `--no-repo` if you'll add a remote yourself.

For public repos, commits use your GitHub no-reply email.

### Development builds: edit code and see it instantly

```bash
ios-cloud run dev             # only needed again when you add or upgrade a native library
npx expo start                # every day: edit code, the app reloads on your iPhone
```

If your iPhone can't reach your PC (for example, the PC is on the iPhone's hotspot), use `npx expo start --tunnel`.

### Commands

| Command | What it does |
|---|---|
| `ios-cloud init` | Set up the current project (options: `--private`, `--no-dev-client`, `--no-repo`, `--force`, `--dry-run`) |
| `ios-cloud run [release\|dev]` | Build in the cloud, then install (and launch) |
| `ios-cloud build [release\|dev]` | Build in the cloud and download the `.ipa` to `build/ios/` |
| `ios-cloud wait <run-id>` | Resume watching a build (after Ctrl+C or a dropped connection) and download it |
| `ios-cloud install [file.ipa]` | Install the latest `.ipa` again, also to **renew it every 7 days** |
| `ios-cloud doctor` | Check the setup |
| `ios-cloud update` | Update this toolkit, and offer to refresh the project's workflow |

The cloud Mac builds what you have **committed**, so commit your changes before `ios-cloud build`. You can also start a build on github.com: **Actions → iOS build → Run workflow**.

## Limits of a free Apple ID

- Apps stop opening after **7 days**. Plug in the iPhone and run `ios-cloud install` to renew. Your app data stays.
- At most **3** sideloaded apps on the iPhone at once, and about **10 new bundle IDs per week**.
- No push notifications, iCloud, Sign in with Apple, or other paid-only capabilities.

## GitHub limits

- **Public repos:** free, unlimited Mac minutes.
- **Private repos:** about 2,000 free minutes a month, and Mac minutes count 10×. That's roughly 200 Mac minutes, or 8–12 native builds. With dev builds you rarely need a native rebuild.
- Each build's `.ipa` is kept on GitHub for 7 days.

## Settings for unusual projects

Edit the `env:` block at the top of `.github/workflows/ios-build.yml` in your project:

- `IOS_SCHEME`: the Xcode scheme to build (default: auto-detect).
- `IOS_XCODE`: a specific Xcode version on the runner, for example `26.6`.
- `NODE_VERSION`: the Node.js version, used if there's no `.nvmrc`.

Environment variables for the `ios-cloud` command:

- `IOS_CLOUD_INSTALLER`: `xtool`, `plumesign`, `wsl`, or `manual` (default: the first one found).
- `IOS_CLOUD_PLUMESIGN`: the path to plumesign if it isn't on your PATH.
- `IOS_CLOUD_RETRIES`: how often to retry network commands (default 3).

## Troubleshooting

- **Build failed:** `ios-cloud` prints the last lines of the failing step and a link to the full log. The full Xcode log is also attached to the run as `xcodebuild-log-*`.
- **"The job was not started because your account is locked due to a billing issue":** GitHub blocks *all* Actions, even free ones on public repos, until the billing problem is fixed. Open https://github.com/settings/billing and fix the payment method, pay or cancel the overdue charge, or cancel any paid plan or trial. Then start the build again.
- **`refusing to allow an OAuth App to create or update workflow`:** run `gh auth refresh -s workflow`.
- **The app won't open:** turn on Developer Mode and trust your Apple ID (see above), or the 7 days ran out (run `ios-cloud install`).
- **Windows: plumesign can't ask for input in Git Bash:** ios-cloud runs it through `winpty` automatically. If it still fails, run the same command in Windows Terminal.
