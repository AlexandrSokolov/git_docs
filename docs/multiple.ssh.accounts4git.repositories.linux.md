## Multiple ssh accounts for different git repositories on Linux

### 1. Generate rsa key, one key per git repository (GitHub, GitLab):

```bash
ssh-keygen -t rsa -b 4096 -C "${email}" -f /home/${user}/.ssh/id_rsa_gitlab
ls -al ~/.ssh
-rw-------  1 ${user} ${user} 3243 Nov 11 13:00 id_rsa_gitlab
-rw-r--r--  1 ${user} ${user}  747 Nov 11 13:00 id_rsa_gitlab.pub
```
For instance: GitLab ssh key for `${user}=alex` system user:
```bash
ssh-keygen -t rsa -b 4096 -C "alexandr.sokolov@uptempo.io" -f /home/alex/.ssh/id_rsa_bm_gitlab
```
Private GitHub ssh key for `${user}=alex` system user:
```bash
ssh-keygen -t rsa -b 4096 -C "sav.public@yandex.com" -f /home/alex/.ssh/id_rsa_github
```
Check results:
```bash
ls -al ~/.ssh
...
-rw-------  1 alex alex 3389 Aug 29 20:46 id_rsa_bm_gitlab
-rw-r--r--  1 alex alex  747 Aug 29 20:46 id_rsa_bm_gitlab.pub
-rw-------  1 alex alex 3389 Aug 29 20:46 id_rsa_gitlab
-rw-r--r--  1 alex alex  753 Aug 29 20:46 id_rsa_gitlab.pub
```

### 2. Upload the generated rsa public keys to your git account configuration.

Copy the **public** key (`.pub`) to the clipboard, then paste it into your git host's SSH-keys page.

```bash
xclip -sel clip < ~/.ssh/id_rsa_bm_gitlab.pub   # X11; on Wayland use: wl-copy < ~/.ssh/id_rsa_bm_gitlab.pub
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

### 3. Multiple accounts on the same host (e.g. two GitHub accounts)

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
