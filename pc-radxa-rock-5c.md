## Installation and configuration

### Install

- https://www.armbian.com/

### Setup

#### ssh

- copy ssh keys from backup to `~/.ssh/`

#### gpg

- copy gpg keys from backup to `~/`

```
$ gpg --import ~/private.gpg
$ gpg --list-secret-keys --keyid-format=long
$ nano ~/.gnupg/gpg-agent.conf
$ cat ~/.gnupg/gpg-agent.conf
default-cache-ttl 3600
allow-loopback-pinentry
pinentry-program /usr/bin/pinentry-curses
$ gpg-connect-agent reloadagent /bye
$ rm ~/private.gpg
$ echo "GPG_TTY=$(tty)" >> ~/.bashrc
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
