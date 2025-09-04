
### Multiple ssh accounts for different git repositories

1. Generate rsa key, one key per git repository (GitHub, GitLab):
    ```bash
    ssh-keygen -t rsa -b 4096 -C "${email}" -f /home/${user}/.ssh/id_rsa_gitlab 
    ls -al ~/.ssh
    -rw-------  1 ${user} ${user} 3243 Nov 11 13:00 id_rsa_gitlab
    -rw-r--r--  1 ${user} ${user}  747 Nov 11 13:00 id_rsa_gitlab.pub
    ```
   Uptempo GitLab ssh key for `${user}=alex` system user:
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

2. Upload the generated rsa public keys to your git account configuration.

   - [Login](https://gitlab.dev.brandmaker.com) with your gitlab account as `alexandr.sokolov` with your domain password and VPN enabled.
   - Copy ssh key into buffer:
       ```bash
       xclip -sel clip < ~/.ssh/id_rsa_bm_gitlab.pub
       ```
   - Install the `id_rsa_bm_gitlab.pub` public key into [SSH keys](https://gitlab.dev.brandmaker.com/-/profile/keys).
   - Test the connection (pathphrase will be asked)
       ```bash
       ssh -T git@gitlab.dev.brandmaker.com
       Welcome to GitLab, Alexandr Sokolov!
       ```
   - [Login](https://github.com/AlexandrSokolov) with your private gitlab account.
   - Copy ssh key into buffer: `$ xclip -sel clip < ~/.ssh/id_rsa_github.pub`
   - Install the `id_rsa_github.pub` public key into [SSH keys](https://github.com/settings/keys).
   - Test the connection (pathphrase will be asked)
       ```bash
       ssh -T git@github.com
       Hi AlexandrSokolov! You've successfully authenticated, but GitHub does not provide shell access.
       ```

3. Enable multiple git repositories and git accounts support:

    Note: this step might be not required. On the system multiple ssh accounts function correctly without having this file.
    
    Create `~/.ssh/config` with the following content:
    ```
    #asokolov private account on github.com
    Host github.com
        HostName github.com
        User asokolov
        IdentityFile ~/.ssh/id_rsa_github
    
    #asokolov Uptempo account
    Host gitlab.dev.brandmaker.com
        HostName gitlab.dev.brandmaker.com
        User asokolov
        IdentityFile ~/.ssh/id_rsa_bm_gitlab
    ```

4. Configure `user.name` and `user.email` git settings.

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
