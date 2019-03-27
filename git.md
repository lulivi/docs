# Usefull commands of git.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Usefull commands of git.](#usefull-commands-of-git)
    - [Configuration](#configuration)
    - [Pulling with unstaged/uncommited changes](#pulling-with-unstageduncommited-changes)
    - [Branches](#branches)
        - [Create branch from local](#create-branch-from-local)
        - [Sync local branch with remone one](#sync-local-branch-with-remone-one)
        - [Merge with other branch](#merge-with-other-branch)
        - [Delete remote/local branch](#delete-remotelocal-branch)
    - [Solving wrong commits](#solving-wrong-commits)
        - [Mending commits](#mending-commits)
        - [Forcing new tree to remote](#forcing-new-tree-to-remote)
    - [Reseting changes](#reseting-changes)
        - [Soft reset](#soft-reset)
        - [Hard reset](#hard-reset)
    - [If you mess it up A LOT](#if-you-mess-it-up-a-lot)
    - [Git submodules](#git-submodules)
        - [Create a submodule](#create-a-submodule)
        - [Delete a submodule](#delete-a-submodule)
    - [Cool commits graph](#cool-commits-graph)
    - [Forks](#forks)

<!-- markdown-toc end -->


## Configuration

Basic configuration:

```sh
git config --global user.name "<username>"
git config --global user.email <email>
git config --global core.editor <editor>
```

## Pulling with unstaged/uncommited changes

```sh
git stash save
git pull
git stash pop
```

## Branches

### Create branch from local

```sh
git branch <new_branch_name>
git checkout <new_branch_name>
git push --set-upstream origin <new_branch_name>
```

### Sync local branch with remone one

```sh
git checkout -b <new_branch_name>
# If there is different name between remote and local
git branch --set-upstream-to=origin/<new_remote_branch_name> <new_branch_name>
# Else
git branch -u origin/<new_branch_name>
```

### Merge with other branch

```sh
git checkout <other_branch>
git merge <merging_branch>
```

### Delete remote/local branch

```sh
git push -d <remote_name> <branch_name>
git branch -d <branch_name>
```

## Solving wrong commits

### Mending commits

Last commit:

```sh
git commit --amend
```

If there are more than one commits to ammend and want to do it interactively:

```sh
# Last commit
git rebase -i HEAD

# 2 last commits
git rebase -i HEAD^

# n last commits
git rebase -i HEAD~<number_of_commits>
```

### Forcing new tree to remote

```sh
git push --force
```

### Squashing commits into one

With previous command rebase, you can do:

```sh
git rebase -i <commit_sha_1>
pick <commit_n_sha_1> <commit_n_message>
pick <commit_n-1_sha_1> <commit_n_message>
pick <commit_n-2_sha_1> <commit_n_message>
pick <commit_n-3_sha_1> <commit_n_message>
...
pick <commit_1_sha_1> <commit_n_message>

# <git rebase info>
```

You have to change pick for squash:

```sh
git rebase -i <commit_sha_1>
pick <commit_n_sha_1> <commit_n_message>
squash <commit_n-1_sha_1> <commit_n_message>
squash <commit_n-2_sha_1> <commit_n_message>
squash <commit_n-3_sha_1> <commit_n_message>
...
squash <commit_1_sha_1> <commit_n_message>

# <git rebase info>
```

## Reseting changes

### Soft reset

Does not touch the index file or the working tree at all

```sh
git reset --soft <commit>
```

### Hard reset

Resets the index and working tree. Any changes to tracked files in the working
tree since <commit> are discarded

```sh
git reset --hard <commit/tree-ish>
```

Ej:

```sh
git reset --hard origin/HEAD
git reset --hard d771744
```

## If you mess it up A LOT

Check local repository updates with

```sh
git reflog
```

Then simply get back in time

```sh
git reset --hard HEAD@{<reflog_number>}
```

## Git submodules

### Create a submodule

To create a submodule:

```sh
git submodule add <submodule_url> [in_repo_submodule_path]
```

### Delete a submodule

To delete it:


1. Remove config entries:

   ```sh
   git config -f .git/config --remove-section submodule."<submodulename>"
   git config -f .gitmodules --remove-section submodule."<submodulename>"
   ```
2. Remove directory from index:

    ```sh
    git rm --cached <submodulepath>
    ```

3. Commit

4. Delete unused files:

    ```sh
    rm -rf <submodulepath>
    rm -rf .git/modules/<submodulename>
    ```

## Cool commits graph

```sh
git log --graph [--oneline] [--all]
```

## Forks

1. Fork the repository on github

1. Clone your forked repository

   ```sh
   git clone https://github.com/YOUR_USERNAME/YOUR_FORK.git
   ```

1. Add the upstream

   ```sh
   git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
   ```

1. Check with if everything is correct

   ```sh
   git remote -v
   origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
   origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
   upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
   upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
   ```

1. Download the original repository changes after pushing to your fork

    ```sh
    git pull upstream <branch>
    ```

1. Push and make a pull request
