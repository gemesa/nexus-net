## Installation and configuration

### Setup

#### ssh

- copy ssh keys from backup to `~/.ssh/`

```
$ ssh-add --apple-use-keychain ~/.ssh/id_ed25519_github
```

```
$ nano ~/.ssh/config
$ cat ~/.ssh/config
Host *
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/id_ed25519_github
```

#### gpg

- copy gpg keys from backup to `~/`

```
$ gpg --import ~/private.gpg
$ gpg --list-secret-keys --keyid-format=long
$ nano ~/.gnupg/gpg-agent.conf
$ cat ~/.gnupg/gpg-agent.conf
default-cache-ttl 3600
$ gpg-connect-agent reloadagent /bye
$ rm ~/private.gpg
```

- double-check `GPG_TTY` in `.zshrc` and add it if necessary

```
$ echo "GPG_TTY=$(tty)" >> ~/.zshrc
```

#### git

```
$ git config --global user.name "Andras Gemes"
$ git config --global user.email "gemesa@protonmail.com"
$ git config --global commit.gpgsign true
$ git config --global user.signingkey <key ID>
$ git config --global core.editor "nano"
$ git config --global --list
$ git config --global diff.tool bc
$ git config --global difftool.bc.path "/Applications/Beyond Compare.app/Contents/MacOS/BCompare"
$ git config --global difftool.prompt false
```

Note: [Beyond Compare](https://www.scootersoftware.com/download) has to be installed manually.

#### keyboard layout

```
$ git clone https://github.com/zaki/mac-hun-keyboard
$ cp mac-hun-keyboard-master/Hungarian_Win.keylayout ~/Library/Keyboard\ Layouts/
```

- reboot and select the layout under the keyboard settings

### brew

- https://brew.sh/

#### Karabiner

```
$ brew install --cask karabiner-elements
```

- `left_control` —> `left_command`
- `left_command` —> `left_control`

#### Rectangle

```
$ brew install --cask rectangle
```

- top left —> top left sixth
- bottom left —> bottom left sixth
- Launch on login

#### System

- System Settings —> Mouse —> disable Natural scrolling
- System Settings -> Keyboard -> Text Input -> Input Sources -> Edit... -> disable "Add period with double-space"
- Night light —> custom —> from 02:00 to 01:59

#### Terminal

- install Nord theme: https://github.com/nordtheme/terminal-app
- `brew install font-fire-code`
- set Fira code and font size 12

#### LLVM

```
$ brew install llvm
$ sudo ln -s /opt/homebrew/opt/llvm/bin/clang-format /usr/local/bin/clang-format
$ sudo ln -s /opt/homebrew/opt/llvm/bin/clang-tidy /usr/local/bin/clang-tidy
$ sudo ln -s /opt/homebrew/opt/llvm/bin/opt /usr/local/bin/opt
$ sudo ln -s /opt/homebrew/opt/llvm/bin/llc /usr/local/bin/llc
$ sudo ln -s /opt/homebrew/opt/llvm/bin/llvm-config /usr/local/bin/llvm-config
```

Alternatively, add `/opt/homebrew/opt/llvm/bin` to the path. This will shadow the preinstalled LLVM tools.

#### Docker

```
$ brew install --cask docker
```

#### VS Code

```
$ brew install --cask visual-studio-code
```

#### Xcode

- install from App Store
- [disable iOS simulator warnings](https://www.reddit.com/r/SwiftUI/comments/1dmukgm/how_to_silence_xcode_debug_output/): `Product` --> `Scheme` --> `Edit Scheme...` --> `Run` --> `Arguments` --> `Environment Variables` --> add: `OS_ACTIVITY_MODE=disable`

#### Android Studio

- https://developer.android.com/studio
- update `~/.zshrc`:
  ```
  export ANDROID_HOME=$HOME/Library/Android/sdk
  export PATH=$PATH:$ANDROID_HOME/emulator
  export PATH=$PATH:$ANDROID_HOME/platform-tools
  export PATH=$PATH:$ANDROID_HOME/build-tools/36.1.0
  ```
- right click on toolbar --> `Add Action to Main Toolbar` --> `Back / Forward`

#### WireGuard

- copy config from backup
- install from App Store

#### Ente Auth

- install from App Store

#### Dock

Remove the following apps from Dock:

- Safari
- Messages
- Mail
- Maps
- Photos
- FaceTime
- Contacts
- Reminders
- Freeform
- TV
- Music

Add the following apps to Dock:

- Firefox
- VS Code
- Xcode
- Terminal
- System Information
- WireGuard
- Ente Auth
- Beyond Compare
- Docker Desktop

#### Other apps

- refer to [pc-thinkpad-l580.md](pc-thinkpad-l580.md)
