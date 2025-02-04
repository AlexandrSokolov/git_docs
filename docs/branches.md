* List of local branches: `$ git branch`
* List of all branches including remote `$ git branch -a`
* [Create local branch from remote](#create-local-branch-from-remote)
* [Create local branch from remote from specific commit](#create-local-branch-from-remote-from-specific-commit)
* [Create local branch from the current local one](#create-local-branch-from-the-current-local-one)
* [Find a remote branch tracked connected to local](#find-a-remote-branch-tracked-by-local)
* [Connect a local branch to the remote](#connect-a-local-branch-to-the-remote)
* [Switch between branches](#switch-between-branches)


#### Create local branch from remote

**Note**: creation from a remote branch makes only sense if you are going to push into the same remote branch.
As a result, to make it more clear, it makes sense to name both remote and local branches in the same way.

If you want to copy one branch, apply changes and push into another remote one, 
create your local branch from another local one, but not from a remote.

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

#### What happens if you create a local branch from a remote one and try to push into another remote?

```bash
$ git branch -a
* TO_BRANCH
  remotes/origin/TO_BRANCH

$ git checkout -b FROM_BRANCH remotes/origin/TO_BRANCH
Branch 'FROM_BRANCH' set up to track remote branch 'TO_BRANCH' from 'origin'.
Switched to a new branch 'FROM_BRANCH'

$ git push -u origin remotes/origin/FROM_BRANCH
error: src refspec remotes/origin/FROM_BRANCH does not match any
error: failed to push some refs to 'git@github.com:AlexandrSokolov/git_test.git'

$ git branch -vv
* FROM_BRANCH d64c99c [origin/TO_BRANCH] Initial commit
  TO_BRANCH   d64c99c [origin/TO_BRANCH] Initial commit
```

In this example you try to push into another remote `FROM_BRANCH` branch, but it does not exist.
You local branch was copied from the remote, as a result both local branches refer to the same `origin/TO_BRANCH` remote.

### Create local branch from remote from specific commit

1. Identify the Commit ID: Ensure that you have the commit ID you want to create the new branch from.

  In IntellijIDEA, right click on the commit and choose `Copy revision number`.
  Commit ID looks as: `96deb096f3ab0f0267b71776689bc31706623318`

2. Checkout the `${commitId}` commit to the new `${localBranchName}` local branch:

```bash
git checkout -b ${localBranchName} ${commitId}
```

3. Create a remote branch based on your new local branch:
```bash
$ git push -u origin BMSUPPORT-34204_sftp_timeout
```

Example:
```bash
$ git branch -a | grep sftp
$ git branch
* develop
$ git checkout -b BMSUPPORT-34204_sftp_timeout 96deb096f3ab0f0267b71776689bc31706623318
Switched to a new branch 'BMSUPPORT-34204_sftp_timeout'
$ git branch
* BMSUPPORT-34204_sftp_timeout
  develop
$ git push -u origin BMSUPPORT-34204_sftp_timeout
$ git branch -vv
* BMSUPPORT-34204_sftp_timeout 96deb09 [origin/BMSUPPORT-34204_sftp_timeout] Solving deletion problem
```

Note: you might meet the error:
```bash
git push -u origin ...
remote: You are not allowed to push code to this project.
fatal: unable to access
returned error: 403
```
Check the connection string:
```bash
$ git remote -vv
origin	git@... (fetch)
origin	git@... (push)
```
Make sure it starts with `git@`, but not with `https`:
```bash
$ git remote -vv
origin	https://... (fetch)
origin	https://... (push)
```
Update the connection to:
```bash
git remote set-url origin git@...
```


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

Works, only if you created your local branch, not from another remote one.

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
