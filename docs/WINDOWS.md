# Windows setup guide (Git Bash, no WSL)

This guide takes a Windows 10/11 PC from nothing to building and installing your app on an iPhone. You do it **once per PC**. After that, each project needs only `ios-cloud init` once, and `ios-cloud run` from then on.

> **Tested on Windows.** Every push to this repo runs [the test workflow](../.github/workflows/test.yml) on a real Windows machine in Git Bash. It checks the installer, a new terminal finding `ios-cloud`, `doctor`, `init`, downloading a real iPhone build, and **plumesign.exe opening that build**. The only step that can't be tested there is the USB install itself, because GitHub has no iPhone plugged in.

## 1. Install the programs

Open **PowerShell** (Start menu → type `PowerShell`) and run:

```powershell
winget install --id Git.Git -e
winget install --id OpenJS.NodeJS.LTS -e
winget install --id GitHub.cli -e
winget install --id Microsoft.WindowsTerminal -e
```

Windows 11 already has Windows Terminal. If winget says something is already installed, that's fine.

Then install **iTunes from Apple's website** (https://support.apple.com/en-us/106372). Impactor needs iTunes' iPhone USB driver, called "Apple Mobile Device Service". You never have to open iTunes.

- Prefer the download from apple.com over the Microsoft Store version. The Store version doesn't register "Apple Mobile Device Service" as a normal Windows service, and Impactor's docs tell you to check that service.
- If plumesign later won't start, install the latest **Microsoft Visual C++ Redistributable (x64)**.

## 2. Use Windows Terminal, not the old "Git Bash" window

Git Bash's default window (mintty) can't show questions from Windows programs like `gh` and `plumesign`. Their password and code prompts just hang. **GitHub's own docs** say the same: `gh help mintty`.

Open **Windows Terminal**, click the **˅** next to the tab, and choose **Git Bash**. Optional: Settings → Startup → Default profile → **Git Bash**. Every command below runs there.

(If you do use the old Git Bash window, `ios-cloud` automatically runs plumesign through `winpty`. For `gh`, type `winpty gh auth login`.)

## 3. Install plumesign (the Windows iPhone installer)

plumesign is the command-line tool of [Impactor](https://github.com/claration/Impactor). It signs the app with your free Apple ID and installs it over USB, doing on Windows what xtool does on Linux.

```bash
mkdir -p ~/bin
curl -fL https://github.com/claration/Impactor/releases/latest/download/plumesign-windows-x86_64.exe -o ~/bin/plumesign.exe
plumesign --version
```

On an ARM PC, use `plumesign-windows-arm64.exe` instead. If Windows Defender flags it, choose **More info → Run anyway**. Impactor is open source.

## 4. Install ios-cloud

```bash
git clone https://github.com/Stephy020/ios-cloud-build ~/ios-cloud-build
~/ios-cloud-build/install.sh
```

Close the tab and open a new Git Bash tab. `ios-cloud version` should now work, in any folder.

## 5. Log in (once)

```bash
gh auth login -s workflow
```

Pick **GitHub.com → HTTPS → Yes** (authenticate Git) **→ Login with a web browser**, then enter the code it shows. Then log in to Apple:

```bash
plumesign account login -u your-apple-id@example.com
```

Type your Apple ID password, and the 6-digit code if your iPhone shows one. Any free Apple ID works.

## 6. Prepare the iPhone (once)

1. Plug the iPhone in with a USB cable and unlock it. Tap **Trust This Computer** and enter your passcode.
2. Turn on **Settings → Privacy & Security → Developer Mode**. The iPhone restarts.

Then check everything:

```bash
ios-cloud doctor
```

You want ✓ for git, Node.js, GitHub, and "Installer: plumesign". If it says the USB driver is missing, see the troubleshooting table below.

## 7. Build and install an app

An existing project, for example MyExpoApp:

```bash
cd ~
git clone https://github.com/Stephy020/MyExpoApp
cd MyExpoApp
npm install
ios-cloud run release
```

A new project, once only:

```bash
cd my-new-app
ios-cloud init
ios-cloud run release
```

When plumesign asks, pick your team and your iPhone with the arrow keys and Enter. After the first install, on the iPhone open **Settings → General → VPN & Device Management →** your Apple ID **→ Trust**. Then open the app.

Every 7 days, plug in the iPhone and run `ios-cloud install` to renew the app. No new cloud build is needed.

## Linux and Windows on the same iPhone

xtool (Linux) and plumesign (Windows) give the app different IDs:

| Installed from | App ID on the iPhone |
|---|---|
| Linux (xtool) | `XTL-<team ID>.com.you.app` |
| Windows (plumesign) | `com.you.app.<team ID>` |

So installing from both puts **two copies** of the app on the phone. They have separate data, and each counts toward the free limit of 3 apps. Pick one machine to install each app from. Building with `ios-cloud build` from either machine is fine.

## Troubleshooting

| Problem | Fix |
|---|---|
| `ios-cloud: command not found` | Open a new Git Bash tab, or run `source ~/.bashrc`. |
| `gh auth login` or plumesign hangs and shows nothing | You're in the old Git Bash window. Use Windows Terminal (step 2), or prefix with `winpty`. |
| `$'\r': command not found` | Something converted the scripts to Windows line endings. Run `rm -rf ~/ios-cloud-build` and clone it again (step 4). |
| doctor: "Apple's iPhone USB driver isn't installed" | Install iTunes from apple.com (step 1), then restart the PC. |
| plumesign: "No devices connected" | Unlock the iPhone, tap Trust, and try another cable or USB port. Then restart the service in PowerShell **as Administrator**: `net stop "Apple Mobile Device Service"` and `net start "Apple Mobile Device Service"`. Close iTunes and other sideloading tools first. |
| plumesign: "No account selected" | Run `plumesign account login -u your-apple-id@example.com`. |
| plumesign doesn't start at all | Install the Microsoft Visual C++ Redistributable (x64). |
| The app won't open on the iPhone | Turn on Developer Mode, trust your Apple ID (step 7), or renew after 7 days with `ios-cloud install`. |
| The build fails on GitHub | ios-cloud prints GitHub's message and the last log lines. "Account locked due to a billing issue" means: update your payment method at https://github.com/settings/billing/payment_information. Nothing is charged for public repos. |
