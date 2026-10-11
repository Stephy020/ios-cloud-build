# Skydrop ☁️📱

**Build your iPhone app in the cloud and drop it onto your phone. No Mac and no paid Apple Developer account, from Linux or Windows.**

[![test](https://github.com/Stephy020/skydrop/actions/workflows/test.yml/badge.svg)](https://github.com/Stephy020/skydrop/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

```bash
skydrop init          # once per project
skydrop run release   # build in the cloud, install on your iPhone, launch it
```

Skydrop is for Expo and React Native developers who want to try their app on a real iPhone without owning a Mac or paying Apple $99 a year.

## How it works

```
   your PC                       GitHub's free Mac                   your PC  ──USB──▶  iPhone
┌────────────────┐  git push   ┌──────────────────────┐  download  ┌──────────────────────────┐
│ skydrop run    │ ──────────▶ │ expo prebuild        │ ─────────▶ │ sign with your FREE      │
│                │             │ pod install          │   .ipa     │ Apple ID, then install   │
│                │             │ xcodebuild, UNSIGNED │            │ (xtool / Impactor)       │
└────────────────┘             └──────────────────────┘            └──────────────────────────┘
```

1. **Your code goes to GitHub.** Skydrop pushes your project and starts a build on one of GitHub's macOS machines. They're free for public repos.
2. **The Mac builds an unsigned app.** It runs Xcode with code signing turned off, so it needs no Apple account or certificates.
3. **Your PC signs and installs it.** Skydrop downloads the `.ipa` and signs it with your free Apple ID, the same "personal team" Xcode uses, then installs it over USB:
   - **Linux:** [xtool](https://xtool.sh)
   - **Windows:** [Impactor's plumesign](https://github.com/claration/Impactor)

## Features

- **One command:** `skydrop run` pushes, builds, waits, downloads, installs and launches.
- **Any Expo project**, and bare React Native projects that have an `ios/` folder.
- **Live development builds:** install once, then edit code and see it on your phone in seconds with `npx expo start`.
- **Linux bash and Windows Git Bash**, no WSL needed. Every change is [tested on real Windows and Linux machines](.github/workflows/test.yml).
- **Made for flaky networks:** retries, resumable waits (`skydrop wait`), and GitHub's error messages shown in your terminal.
- **Careful by default:** public repos commit with your GitHub no-reply email, generated folders stay out of git, and nothing is ever signed in the cloud.

## Quick start

### 1. Install Skydrop (once per computer)

You need git, [Node.js](https://nodejs.org) 22+, and the [GitHub CLI](https://cli.github.com) (`gh`).

```bash
git clone https://github.com/Stephy020/skydrop ~/skydrop
~/skydrop/install.sh
gh auth login -s workflow
```

Then set up the part that installs apps on the iPhone:

| | What to install |
|---|---|
| **Linux** | [xtool](https://xtool.sh), then `xtool setup` to log in with your Apple ID |
| **Windows** | iTunes (for the USB driver) and `plumesign.exe`. Follow the **[Windows guide](docs/WINDOWS.md)**. It's step by step, and it explains why you should use Windows Terminal instead of the old Git Bash window |

### 2. Prepare your iPhone (once)

- **Settings → Privacy & Security → Developer Mode → On** (the iPhone restarts).
- Plug it in by USB, unlock it, and tap **Trust This Computer**.

Check that everything's ready:

```bash
skydrop doctor
```

### 3. Build and install

```bash
cd my-expo-app
skydrop init          # adds the workflow, expo-dev-client, and creates the GitHub repo
skydrop run release   # about 7–20 minutes the first time
```

After the first install: **Settings → General → VPN & Device Management →** your Apple ID **→ Trust**.

## Everyday use

### Development builds: change code and see it instantly

```bash
skydrop run dev       # only needed again when you add or upgrade a native library
npx expo start        # every day: save a file, and the app on your iPhone reloads
```

If your iPhone can't reach your PC (for example, the PC is on the iPhone's hotspot), use `npx expo start --tunnel`.

### Commands

| Command | What it does |
|---|---|
| `skydrop init` | Set up the current project (options: `--private`, `--no-dev-client`, `--no-repo`, `--force`, `--dry-run`) |
| `skydrop run [release\|dev]` | Build in the cloud, then install (and launch) |
| `skydrop build [release\|dev]` | Build in the cloud and download the `.ipa` to `build/ios/` |
| `skydrop wait <run-id>` | Resume watching a build (after Ctrl+C or a dropped connection) and download it |
| `skydrop install [file.ipa]` | Install the latest `.ipa` again, also to **renew it every 7 days** |
| `skydrop doctor` | Check the setup |
| `skydrop update` | Update Skydrop, and offer to refresh the project's workflow |

- **`release`** is a standalone app with the JavaScript inside.
- **`dev`** is an Expo development build that loads your code live from `npx expo start`.

The cloud Mac builds what you have **committed**, so commit before `skydrop build`. You can also start a build on github.com: **Actions → iOS build → Run workflow**.

## Limits

Most of these come from Apple's free accounts and GitHub's free plan, not from Skydrop.

| Limit | Details |
|---|---|
| **7 days** | Apps signed with a free Apple ID stop opening after 7 days. Plug in and run `skydrop install` to renew. Your app data stays |
| **3 apps** | At most 3 sideloaded apps on the iPhone at once, and about 10 new app IDs per week |
| **Capabilities** | No push notifications, iCloud, Sign in with Apple, or other paid-only features |
| **Sharing** | The signed app runs only on iPhones you install it on. To send it to people over the internet (TestFlight or the App Store), you need the paid Apple Developer Program |
| **GitHub minutes** | Public repos: free and unlimited. Private repos: about 200 macOS minutes a month (roughly 8–12 builds) |
| **One installer per app** | xtool and plumesign give the app different IDs (`XTL-<team>.<id>` vs `<id>.<team>`). Installing from both gives you two copies |

## Settings

In your project's `.github/workflows/ios-build.yml` (the `env:` block):

- `IOS_SCHEME`: the Xcode scheme to build (default: auto-detect).
- `IOS_XCODE`: a specific Xcode version on the runner, for example `26.6`.
- `NODE_VERSION`: the Node.js version, used if there's no `.nvmrc`.

Environment variables for the `skydrop` command:

- `SKYDROP_INSTALLER`: `xtool`, `plumesign`, `wsl`, or `manual` (default: the first one found).
- `SKYDROP_PLUMESIGN`: the path to plumesign if it isn't on your PATH.
- `SKYDROP_RETRIES`: how often to retry network commands (default 3).

## Troubleshooting

- **Build failed:** Skydrop prints GitHub's message, the last lines of the failing step, and a link to the full log. The full Xcode log is also attached to the run as `xcodebuild-log-*`.
- **"The job was not started because your account is locked due to a billing issue":** GitHub blocks all Actions, even free ones, until this is fixed. Update your payment method at https://github.com/settings/billing/payment_information; nothing is charged for public repos. Wait up to 24 hours, or ask GitHub Support.
- **`refusing to allow an OAuth App to create or update workflow`:** run `gh auth refresh -s workflow`.
- **The app won't open:** turn on Developer Mode and trust your Apple ID (see above), or the 7 days ran out (run `skydrop install`).
- **Windows problems:** see the troubleshooting table in the [Windows guide](docs/WINDOWS.md#troubleshooting).

## FAQ

**Is this allowed?** Skydrop uses Apple's free personal development signing, the same thing Xcode does when you run an app on your own phone with a free Apple ID. It's meant for testing your own apps on your own devices, not for distributing apps.

**Can I publish to the App Store with this?** No. App Store and TestFlight distribution need the paid Apple Developer Program. Skydrop is for building and testing.

**What about Android?** Skydrop only handles iOS. For Android, a normal `npx expo run:android` or Gradle build works on Linux and Windows directly.

**Does my code have to be public?** No. Use `skydrop init --private`. Private repos just get fewer free macOS minutes.

## Credits

Skydrop is the glue. These projects do the hard work:

- [xtool](https://github.com/xtool-org/xtool): signing and installing on Linux.
- [Impactor / plumesign](https://github.com/claration/Impactor): signing and installing on Windows.
- [Expo](https://expo.dev): `expo prebuild` and development builds.
- [GitHub Actions](https://github.com/features/actions): the free macOS machines.

## License

[MIT](LICENSE) © 2026 Stephen Abudu (Stephy020).

Skydrop isn't affiliated with Apple, GitHub or Expo. Apple, iPhone, iOS and Xcode are trademarks of Apple Inc.
