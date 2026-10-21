# react-setup

# Part 1: Full setup on a fresh Ubuntu

Paste the blocks one at a time, in order, and wait for each to finish.

## 1. Base packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget unzip build-essential procps file ca-certificates gnupg htop
```

## 2. File watcher limits

```bash
printf 'fs.inotify.max_user_watches=524288\nfs.inotify.max_user_instances=512\n' | sudo tee /etc/sysctl.d/99-expo-watchers.conf
sudo sysctl --system
```

## 3. Node.js

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install --lts
node -v && npm -v
```

## 4. Java 17

```bash
sudo apt install -y openjdk-17-jdk
```

## 5. Environment variables (run once only)

```bash
cat >> ~/.bashrc << 'EOF'

# >>> expo-dev >>>
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$JAVA_HOME/bin:$PATH:$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin
# <<< expo-dev <<<
EOF
source ~/.bashrc
java -version
```

## 6. KVM (fast emulator)

```bash
sudo apt install -y qemu-system-x86 cpu-checker
sudo adduser $USER kvm
kvm-ok
```

You want "KVM acceleration can be used". If not, turn on Intel VT-x or AMD SVM in BIOS.

## 7. Android Studio and the emulator

```bash
sudo snap install android-studio --classic
```

1. Open Android Studio, choose **Standard**, and finish the wizard.
2. Go to **More Actions → SDK Manager → SDK Tools** and tick Platform-Tools, Build-Tools, Command-line Tools (latest) and Android Emulator.
3. Go to **Device Manager → Create Device → Pixel → x86_64 image → Show Advanced Settings**. Set Graphics: Hardware, RAM: 3072 MB, VM heap: 384 MB, Internal storage: 8 GB, CPU cores: 4, SD card: none.

```bash
adb version
emulator -list-avds
```

If the emulator crashes or shows a black screen, close it and edit `~/.android/avd/NAME.avd/config.ini`. Set `hw.gpu.mode=swiftshader_indirect`, then switch back to `host` for normal use.

## 8. Homebrew and Watchman

```bash
sudo apt install -y build-essential procps curl file git
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

```bash
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install watchman
which watchman
watchman --version
```

If `brew install` starts compiling for a long time instead of downloading, press Ctrl+C and run `sudo apt install -y watchman`.

## 9. EAS and Git

```bash
npm install -g eas-cli
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub
```

Paste that last output into GitHub → Settings → SSH keys. Then:

```bash
eas login
```

## 10. VS Code extensions

```bash
code --install-extension dbaeumer.vscode-eslint
code --install-extension esbenp.prettier-vscode
code --install-extension expo.vscode-expo-tools
code --install-extension dsznajder.es7-react-js-snippets
code --install-extension eamodio.gitlens
```

## 11. Log out and back in

Do this now, or reboot. It applies the KVM group and the new `.bashrc` settings.

## 12. Your project

Existing project:

```bash
mkdir -p ~/projects && cd ~/projects
git clone git@github.com:YOU/REPO.git
cd REPO
npm install
```

Restore your `.env` file from your backup. Or a new project:

```bash
mkdir -p ~/projects && cd ~/projects
npx create-expo-app@latest my-app
cd my-app
npx expo install @supabase/supabase-js @react-native-async-storage/async-storage react-native-url-polyfill
npm install @reduxjs/toolkit react-redux @tanstack/react-query
npm install supabase --save-dev
npx supabase login
```

Put your keys in `.env`, and never commit it. Never put the `service_role` key in the app.

```
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

## 13. Final check (inside the project folder)

```bash
node -v
java -version
adb version
watchman --version
npx expo-doctor
npx expo start
```

Press `a` for the emulator, or scan the QR code with Expo Go on your phone.

## Warnings you can ignore

- `npm warn deprecated uuid ...`, `glob`, `rimraf`, `inflight`: old libraries inside Expo and EAS tools. They are harmless.
- `install-scripts ... not yet covered by allowScripts`: you don't need to allow them.
- "New major version of npm available": leave npm as it is.
- Only `npm error` lines are real problems.
- Your 16 GB laptop showing about 14 GB is normal, because integrated graphics reserve some RAM.

---

# Part 2: How to update later

Check what you have first:

```bash
node -v
eas --version
watchman --version
```

## EAS CLI

```bash
npm install -g eas-cli@latest
eas --version
```

## Expo in a project

There is no global Expo CLI to update. The version lives inside each project. From the project folder:

```bash
git add -A && git commit -m "before expo upgrade"
npm install expo@latest
npx expo install --fix
npx expo-doctor
```

`npx expo install --fix` updates your other packages to match the new Expo SDK. Commit first so you can undo it. Major SDK upgrades can change how some libraries work, so if your project works and you're still learning, you can stay on your current version. Also check that Expo Go on your phone supports the new SDK.

## Node.js

```bash
nvm install --lts --reinstall-packages-from=current
nvm alias default 'lts/*'
node -v
```

The `--reinstall-packages-from=current` part carries over your global packages, like `eas-cli`.

## Watchman, Homebrew and Ubuntu

```bash
brew update && brew upgrade watchman
sudo apt update && sudo apt upgrade -y
```

## Android Studio and the emulator

Android Studio, installed with snap, updates itself in the background. You can also run `sudo snap refresh android-studio`. SDK parts update inside Android Studio, under **SDK Manager → SDK Tools**, by ticking "Show Package Details" and the update boxes.

## npm itself

Leave it alone. The npm that comes with your Node LTS is the one Expo is tested with. If you really want to update it, run `npm install -g npm@latest`.

## How often

- **Every few weeks:** `sudo apt update && sudo apt upgrade -y`, and `npm install -g eas-cli@latest`.
- **When you start a new project:** use `npx create-expo-app@latest`.
- **Only when you need a new feature:** upgrade the Expo SDK in an existing project.

If any command gives an `npm error` or another error, paste it here and I'll help.
