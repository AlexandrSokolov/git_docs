* List of local branches: `$ git branch`
* List of all branches including remote `$ git branch -a`
* [Create local branch from remote](#create-local-branch-from-remote)
* [Create local branch from the current local one](#create-local-branch-from-the-current-local-one)
* [Find a remote branch tracked connected to local](#find-a-remote-branch-tracked-by-local)
* [Connect a local branch to the remote](#connect-a-local-branch-to-the-remote)
* [Switch between branches](#switch-between-branches)


#### Create local branch from remote

```bash
git checkout -b ${localBranchName} remotes/origin/${remoteBranchName}
```

Example:
```bash
$ git branch -a
* develop
  ...
  remotes/origin/someRemoteBranch

$ git checkout -b newLocalBranchName remotes/origin/someRemoteBranch
```

[How to cleanly get/copy a remote git branch to local repository](https://stackoverflow.com/questions/11356460/how-to-cleanly-get-copy-a-remote-git-branch-to-local-repository)

#### Create local branch from the current local one

```bash
$ git branch
  someBranch1
  someBranch2
* develop
$ git checkout -b ${newLocalBranchName}
```

#### Find a remote branch tracked by local

Check to which remote branch the current local one is connected (which remote branch is tracked).

```bash
$ git branch -vv
$ git branch -a -vv
$ git remote show ${remote_name}
$ git remote show origin
```

[Find out which remote branch a local branch is tracking](https://stackoverflow.com/questions/171550/find-out-which-remote-branch-a-local-branch-is-tracking)

#### Connect a local branch to the remote

```bash
$ git branch -vv
* jaxrs_resteasy_wf10 c886d8a Added default error logging for auth requests
  master              c886d8a [origin/master] Added default error logging for auth requests

$ git push -u origin jaxrs_resteasy_wf10

$ git branch -vv
* jaxrs_resteasy_wf10 c886d8a [origin/jaxrs_resteasy_wf10] Added default error logging for auth requests
  master              c886d8a [origin/master] Added default error logging for auth requests
```

#### Switch between branches

If  you do not have it as a local branch yet and there is a corresponding remote branch, you get it from remote:
```bash
$ git checkout dse_v6.2
Branch 'dse_v6.2' set up to track remote branch 'dse_v6.2' from 'origin'.
Switched to a new branch 'dse_v6.2'
```

If you already have it as a local branch:
```bash
git checkout develop
Previous HEAD position was 86644e0... Last commit.
Switched to branch 'develop'
Your branch is up-to-date with 'origin/develop'.
```
