
## Multiple ssh accounts for different git repositories on Windows

### 1. Install Git

```powershell
winget install --id Git.Git -e --source winget
```
**Then close this shell and open a new one.** 
PATH is read only when a shell starts, so `git` is invisible in the shell that ran the install. 
Verify in the fresh shell:
```powershell
git --version
```
If `winget` is unavailable, download the installer from https://git-scm.com/download/win and accept the defaults
(they add Git to PATH).

### 2. Make SSH offer the key via ssh-agent

A key with a non-default name (`id_rsa_github`, not `id_rsa`) is **not** tried automatically. 
Without this step, `git clone` and `ssh -T git@github.com` fail with `Permission denied (publickey)` 
even though the key is valid and uploaded. The agent solves this the same way it does on Linux: 
it holds the key and SSH offers it regardless of filename.


The Windows OpenSSH agent ships **disabled**. Enabling it requires **an elevated PowerShell** — open PowerShell via
right-click → **Run as administrator** (title bar must read *Administrator*), or the next command fails with
`Access is denied`.


One-time, in the **elevated** shell:
```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Point Git to use Windows' built-in OpenSSH

Run this in a **normal (non-admin) PowerShell** — `git config --global` writes to *your* user's `~/.gitconfig`.
If you run it in the elevated shell from the previous step, it writes to the administrator's profile instead,
and your own `git` won't pick it up. Close the admin shell first, or open a fresh normal one.

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
git config --global --get core.sshCommand
```
Windows has **two** `ssh.exe` binaries, and they do not share the agent:

- **Windows' built-in OpenSSH** (`C:\Windows\System32\OpenSSH\ssh.exe`) talks to the ssh-agent service you
  just started, which holds your key. This is what a bare `ssh -T git@github.com` uses.
- **Git for Windows bundles its own** `ssh.exe` (under `C:\Program Files\Git\usr\bin\`), and `git` uses that
  one by default. It does **not** talk to the Windows agent, so it offers no key.

The result is that `ssh -T git@github.com` can succeed while `git clone` still fails with 
`Permission denied (publickey)` — same host, same key, different ssh. The config line above forces `git` to use
the built-in OpenSSH, so both go through the one agent that holds your key.

Use forward slashes exactly as shown — Git config treats backslashes as escape characters.

### 3. Generate rsa key, one key per git repository (GitHub, GitLab):

Run in PowerShell (OpenSSH is built in on Windows 10 1809+ / 11). 
Home dir is `C:\Users\<user>`; `~` and `$env:USERPROFILE` both resolve to it.

**First create the `.ssh` directory** — `ssh-keygen` on Windows does not create it for you, and will fail with
`No such file or directory` if it's missing:
```powershell
New-Item -ItemType Directory -Force -Path $env:USERPROFILE\.ssh
```
`-Force` makes this a no-op if the directory already exists, so it is safe to run either way.

Generate the key (the key path is pinned by `-f`, so your current working directory does not matter):
```powershell
ssh-keygen -t rsa -b 4096 -C "sav.public@yandex.com" -f $env:USERPROFILE\.ssh\id_rsa_github
Get-ChildItem ~\.ssh
```
In cmd, use `%USERPROFILE%` instead of `$env:USERPROFILE`, and `dir %USERPROFILE%\.ssh` to list.

Load the key once and test:
```powershell
ssh-add $env:USERPROFILE\.ssh\id_rsa_github
ssh-add -l
```

Note: the Windows agent is supposed to restore loaded keys after a reboot, but this has varied across builds. 
If `ssh-add -l` is empty after a reboot, just rerun the `ssh-add` line.

### 4. Upload the generated rsa public keys to your git account configuration.

Copy the **public** key (`.pub`) to the clipboard, then paste it into your git host's SSH-keys page.

```powershell
Get-Content ~\.ssh\id_rsa_github.pub | clip      # PowerShell
```
In cmd:
```cmd
clip < %USERPROFILE%\.ssh\id_rsa_github.pub
```

Then:

- **GitLab** — [Login](https://gitlab.dev.brandmaker.com) as `alexandr.sokolov` with your domain password and
  VPN enabled. Install `id_rsa_bm_gitlab.pub` into
  [SSH keys](https://gitlab.dev.brandmaker.com/-/profile/keys). Test (passphrase will be asked):
```bash
ssh -T git@gitlab.dev.brandmaker.com
Welcome to GitLab, Alexandr Sokolov!
```
- **GitHub** — [Login](https://github.com/AlexandrSokolov) with your private account. Install
  `id_rsa_github.pub` into [SSH keys](https://github.com/settings/keys). Test (passphrase will be asked):
```bash
ssh -T git@github.com
Hi AlexandrSokolov! You've successfully authenticated, but GitHub does not provide shell access.
```

### Multiple accounts on the same host (e.g. two GitHub accounts) 

TODO, not tested for Windows Yet

The steps above assume one account per host. That breaks down when two accounts share the *same* host — 
e.g. a personal GitHub account and a client/second GitHub account, both under `github.com`. 
SSH and Git have no way to tell which key you mean just from `git@github.com:...`, since the hostname is identical.

**Test each key explicitly against the shared host:**

```bash
ssh -T -i ~/.ssh/id_rsa_github -o IdentitiesOnly=yes git@github.com
Hi AlexandrSokolov! You've successfully authenticated, but GitHub does not provide shell access.

ssh -T -i ~/.ssh/id_rsa_github_sav-public-de -o IdentitiesOnly=yes git@github.com
Hi sav-public-de! You've successfully authenticated, but GitHub does not provide shell access.
```

`-i` picks the key explicitly; `-o IdentitiesOnly=yes` stops SSH from also offering other keys it finds, 
so the test result is unambiguous.


**Permanent solution — select the key by folder, not by editing URLs.**

Git supports conditional config includes based on the repo's location on disk (`includeIf "gitdir:..."`). 
This lets `git clone git@github.com:...` work exactly as copy-pasted, 
with the right key and identity chosen automatically based on *where* you clone it.

1. Create a per-account config file, e.g. `~/.gitconfig-sav-public-de`:

```ini
[core]
    sshCommand = "ssh -i ~/.ssh/id_rsa_github_sav-public-de -o IdentitiesOnly=yes"
[user]
    email = alexandr.sokolov@uptempo.io
    name = asokolov
```

2. Reference it conditionally from the main `~/.gitconfig`, scoped to a folder:

```ini
[includeIf "gitdir:~/projects/allocadia/**"]
    path = ~/.gitconfig-sav-public-de
```

3. Clone as normal, as long as it lands under that folder:

```bash
mkdir -p ~/projects/allocadia
cd ~/projects/allocadia
git clone git@github.com:allocadia/azuqua.git
```

Everything cloned under `~/projects/allocadia/` now uses that SSH key and identity automatically. 
Repos outside that path keep using the default key from step 1/2.

### 4. Enable multiple git repositories and git accounts support (`~/.ssh/config`)

**What this file does:** it lets you assign a specific `IdentityFile` (and other SSH options) per host, 
so SSH doesn't have to guess which key to try. 
Without it, SSH falls back to trying its default keys one by one and uses whichever one the server accepts.

**When it's NOT required — distinct hosts, one key each.**

If every host you connect to (`gitlab.dev.company.com`, `github.com`, etc.) only has *one* of your keys registered on it, 
there's no ambiguity to resolve: SSH tries the key(s) it has, the wrong ones get rejected, the right one succeeds. 
This is why the original two-account setup (GitLab + personal GitHub) worked correctly even without this file — 
each host could only authenticate one key, so the "guessing" always landed correctly by process of elimination.

**Is it ever actually required? No.**

You might reach for `~/.ssh/config` when the same host accepts more than one of your keys 
(two GitHub accounts, both `github.com`). But it doesn't cleanly solve that case: 
`Host` blocks match only on the **hostname**, nothing after it, 
so you cannot have two `Host github.com` blocks disambiguated by org or repo path — SSH never sees that part of the URL. 
The only way to make SSH config distinguish two GitHub accounts is to invent a fake host alias 
(e.g. `Host github.com-sav-public-de`) and then **manually rewrite the clone URL** to use it — 
which also means teaching your machine to resolve that fake name back to `github.com`. 
That's more moving parts, and it defeats the whole point of pasting the URL as-is.

The one thing `~/.ssh/config` genuinely offers is a minor optimisation: 
instead of letting SSH iterate through candidate keys, you pin "this host → this key" up front. 
In practice this buys nothing meaningful.

**Preferred approach in all cases:** the `includeIf "gitdir:..."` method from #3. 
If the repos in a given folder all belong to one account, point that folder at the right key and identity via git config. 
The clone URL stays exactly as copy-pasted, no fake hostnames, 
no URL rewriting, and both the SSH key and `user.name`/`user.email` follow the folder automatically.

**Rule of thumb:**
- One account per host → nothing extra needed; the default key just works.
- Multiple accounts on the *same* host → use `includeIf "gitdir:..."` (see #3). 
  Don't use `~/.ssh/config` host aliases for this — they force manual URL edits and solve nothing 
  that `includeIf` doesn't solve more cleanly.

### 5. Configure `user.name` and `user.email` git settings.

**DO NOT** set global settings like:
```bash
$ git config --global user.email "you@example.com" 
$ git config --global user.name "Your Name"
```
Configure them per project instead. Example for a private GitHub project:
```bash
cd ~/projects/private/project/path
git config user.name asokolov
git config user.email sav.public@yandex.com
git config --list
```
Output:
```
user.name=asokolov
user.email=sav.public@yandex.com
```

Commands for BM GitLab account:
```bash
git config user.name asokolov
git config user.email alexandr.sokolov@uptempo.io
```
