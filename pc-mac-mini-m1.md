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
```

#### keyboard layout

```
$ git clone https://github.com/zaki/mac-hun-keyboard
$ cp mac-hun-keyboard-master/Hungarian_Win.keylayout ~/~/Library/Keyboard\ Layouts/
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

#### System

- System Settings —> Mouse —> disable Natural scrolling
- Night light —> custom —> from 02:00 to 01:59

#### Terminal

- install Nord theme: https://github.com/nordtheme/terminal-app
- `brew install font-fire-code`
- set Fira code and font size 12

#### Other apps

- refer to [pc-thinkpad-l580.md](pc-thinkpad-l580.md)
