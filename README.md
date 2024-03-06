# github-demo/ azure repos
A demo of github

Git - this is a distrubuted system, installer available from git-scm.com for Mac, Linux and Windows.

Default branch name for a git repo is 'master' (created using git init) but can be changed to others as 'main', 'trunk' etc.

## First time adding a project to Git (or creating repo from blank)

### Step 1:
```bash
git init
Initialized empty Git repository in /Users/gs/Documents/azure/gitdemo/.git/
```
git init creates a local .git folder which acts as local DB where all the changes are tracked.

- List global attributes<br>
```bash
(base) gs@GSs-MacBook-Pro gitdemo % git config --global --list 
user.name=V Kumar
user.email=vkcode7@yahoo.com
```

Think there are 3 areas when working with git
1) Local folder(s) where files are
2) Staging area
3) .git repository

- Our test folder has 2 files - FileA.txt and FileB.txt<br>

```bash
(base) gs@GSs-MacBook-Pro gitdemo % ls -l
total 16
-rw-r--r--  1 gs  staff  14 Feb 28 16:28 FileA.txt
-rw-r--r--  1 gs  staff  13 Feb 28 16:28 FileB.txt
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        FileA.txt
        FileB.txt

nothing added to commit but untracked files present (use "git add" to track)<br>
```

### Step 2: Use git add to add the files to staging area

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git add .
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   FileA.txt
        new file:   FileB.txt
```

### Step 3: Next step is to move them from Staging Area to Git repo by using "git commit"

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git commit -m "initial commit"
[master (root-commit) dd8d9ee] initial commit
 2 files changed, 2 insertions(+)
 create mode 100644 FileA.txt
 create mode 100644 FileB.txt
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch master
nothing to commit, working tree clean
```

## Making changes and commiting them
Lets make a change in FileA.txt and go a git status<br><br>

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   FileA.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

Next move it to Staging Area using "git add FileA.txt"

Next commit them:

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git add FileA.txt
(base) gs@GSs-MacBook-Pro gitdemo % git commit -m "Changes made to FileA.txt"
[master 9f2f824] Changes made to FileA.txt
 1 file changed, 1 insertion(+)
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch master
nothing to commit, working tree clean
```

## Going back to previous version of a file

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git log
commit 9f2f824a478cdd4b789cb9dad9d141ecd935aad8 (HEAD -> master)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 17:09:46 2024 -0500

    Changes made to FileA.txt

commit dd8d9eee67195f93c57a1b9f3d9317fe2f200589
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 16:40:44 2024 -0500

    initial commit
(base) gs@GSs-MacBook-Pro gitdemo % git checkout dd8d9eee67195f93c57a1b9f3d9317fe2f200589
Note: switching to 'dd8d9eee67195f93c57a1b9f3d9317fe2f200589'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at dd8d9ee initial commit
(base) gs@GSs-MacBook-Pro gitdemo % git log
commit dd8d9eee67195f93c57a1b9f3d9317fe2f200589 (HEAD)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 16:40:44 2024 -0500

    initial commit
(base) gs@GSs-MacBook-Pro gitdemo % 
```

## Going back to most recent version
If we want to go back to most recent version again after the checkout above using 'git log --all' to see all versions, and then use git checkout

```base
(base) gs@GSs-MacBook-Pro gitdemo % git log --all
commit 9f2f824a478cdd4b789cb9dad9d141ecd935aad8 (master)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 17:09:46 2024 -0500

    Changes made to FileA.txt

commit dd8d9eee67195f93c57a1b9f3d9317fe2f200589 (HEAD)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 16:40:44 2024 -0500

    initial commit
(base) gs@GSs-MacBook-Pro gitdemo % git checkout 9f2f824a478cdd4b789cb9dad9d141ecd935aad8
Previous HEAD position was dd8d9ee initial commit
HEAD is now at 9f2f824 Changes made to FileA.txt
(base) gs@GSs-MacBook-Pro gitdemo % git log
commit 9f2f824a478cdd4b789cb9dad9d141ecd935aad8 (HEAD, master)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 17:09:46 2024 -0500

    Changes made to FileA.txt

commit dd8d9eee67195f93c57a1b9f3d9317fe2f200589
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 16:40:44 2024 -0500

    initial commit
```

## Unstage a change

You make a change to file and run "git add ." to stage the changes, but now you want to unstage that.

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git add .
(base) gs@GSs-MacBook-Pro gitdemo % git status
HEAD detached at 9f2f824
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   FileA.txt

(base) gs@GSs-MacBook-Pro gitdemo % 
```

Here is how you unstage
```bash
(base) gs@GSs-MacBook-Pro gitdemo % git restore --staged FileA.txt
(base) gs@GSs-MacBook-Pro gitdemo % git status
HEAD detached at 9f2f824
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   FileA.txt

no changes added to commit (use "git add" and/or "git commit -a")
(base) gs@GSs-MacBook-Pro gitdemo % git restore FileA.txt
```

You can also do the following to unstage
```bash
(base) gs@GSs-MacBook-Pro gitdemo % git checkout master
Switched to branch 'master'
(base) gs@GSs-MacBook-Pro gitdemo % git branch
* master
(base) gs@GSs-MacBook-Pro gitdemo % git rm --cached FileA.txt
rm 'FileA.txt'
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    FileA.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        FileA.txt

(base) gs@GSs-MacBook-Pro gitdemo % git add .
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch master
nothing to commit, working tree clean
```

## Working with branches

The new features should be worked upon using a separate branch, not the main one. Once everything is developed and tested, only then it should be merged with main.<br>
quick tips:<br>
- creating a branch: git checkout -b FeatureA
- switching to a branch: git chcekout FeatureA
- deleting a branch: git branch --delete FeatureA
- listing branches: git branch

### Creating a branch and commiting code
Using -a option we can create and checkout the branch. All code is then copied to new branch:

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git checkout -b FeatureA
Switched to a new branch 'FeatureA'
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch FeatureA
nothing to commit, working tree clean
(base) gs@GSs-MacBook-Pro gitdemo % git log
commit 9f2f824a478cdd4b789cb9dad9d141ecd935aad8 (HEAD -> FeatureA, master)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 17:09:46 2024 -0500

    Changes made to FileA.txt

commit dd8d9eee67195f93c57a1b9f3d9317fe2f200589
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 16:40:44 2024 -0500

    initial commit
(base) gs@GSs-MacBook-Pro gitdemo % 
```

Lets create a new file FileC.txt, add to staging area and then commit

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git branch
* FeatureA
  master
(base) gs@GSs-MacBook-Pro gitdemo % git add FileC.txt
(base) gs@GSs-MacBook-Pro gitdemo % git commit -m "Added FileC"
[FeatureA bd0f221] Added FileC
 1 file changed, 1 insertion(+)
 create mode 100644 FileC.txt
(base) gs@GSs-MacBook-Pro gitdemo % git log
commit bd0f2217e8a699909e5a4d1b569a0401f060633f (HEAD -> FeatureA)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 21:15:32 2024 -0500

    Added FileC

commit 9f2f824a478cdd4b789cb9dad9d141ecd935aad8 (master)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 17:09:46 2024 -0500

    Changes made to FileA.txt

commit dd8d9eee67195f93c57a1b9f3d9317fe2f200589
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 16:40:44 2024 -0500

    initial commit
```

if we do a git checkout on main branch, you will notice that FileC.txt will disappear as it is not in main. Doing a 'git chcekout FeatureA' will bring it back

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git checkout master
Switched to branch 'master'

(base) gs@GSs-MacBook-Pro gitdemo % ls -l
total 16
-rw-r--r--  1 gs  staff  36 Feb 28 21:03 FileA.txt
-rw-r--r--  1 gs  staff  13 Feb 28 16:28 FileB.txt
(base) gs@GSs-MacBook-Pro gitdemo % 
```

### Merging branches

To merge FeatureA, first chcekout to main and then do a merge:

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git checkout master
Switched to branch 'master'
(base) gs@GSs-MacBook-Pro gitdemo % git merge FeatureA
Updating 9f2f824..bd0f221
Fast-forward
 FileC.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 FileC.txt
```

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git log
commit bd0f2217e8a699909e5a4d1b569a0401f060633f (HEAD -> master, FeatureA)
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 21:15:32 2024 -0500

    Added FileC

commit 9f2f824a478cdd4b789cb9dad9d141ecd935aad8
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 17:09:46 2024 -0500

    Changes made to FileA.txt

commit dd8d9eee67195f93c57a1b9f3d9317fe2f200589
Author: V Kumar <vkcode7@yahoo.com>
Date:   Wed Feb 28 16:40:44 2024 -0500

    initial commit
```

## Remote repo
So far we have been working with local repo,  but in real world devs works with remote repositories such as github or Azure Repos.

So devs do the following:
1. Clone the repo
2. Make local changes
3. Merge local changes back to remote repo

## Push an existing repo that we created locally to remote git repo

This will merge local repo to the remote git repo
```bash
(base) gs@GSs-MacBook-Pro gitdemo % git remote add origin https://github.com/vkcode7/github-demo.git
(base) gs@GSs-MacBook-Pro gitdemo % git push -u origin main
```

If we make any changes now to a file and commit, it will still be in local repo. To push those changes in remote git repo use:<br>
git push -u origin main

Following will set your local to remote/main; all the changes will then be committed to remote

```bash
git branch --set-upstream-to=origin/<branch> main
```

Note that a first time user will have to create a git repo locally using git init, and then publish it on github. However, once published, other users will simply clone and work on it.

## Conflicts
 
Scenario: User1 and User2 are working on same branch (say main). User1 is working on FileA.txt and made changes to it, meanwhile User2 also made a change and pushes it to repo. Now when User1 tries to commit it, error message will show that User1 file is behind and he will have to do a pull. Ater pull he can either accept user2 changes or his own and then can commit again.

### Fast Forward merge
If you create a feature branch and commit changes using fast forward, then changes committed by other users (on their code) wont be in your branch. So always do a pull before commit.

However, lets say you do that in this order:

- create branchA: git checkout -b branchA
- You make change to FileC.txt
- Someone makes a change to ReadMe.md and commit/push in main
- You commit yours changes: git commit -am "testing ff merge"

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch branchA
nothing to commit, working tree clean
(base) gs@GSs-MacBook-Pro gitdemo % git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
(base) gs@GSs-MacBook-Pro gitdemo % git merge --ff-only branchA
Updating 6bd689e..05a574f
Fast-forward
 FileC.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
(base) gs@GSs-MacBook-Pro gitdemo % git log --graph
* commit 05a574f68dc4205daa1a766b35047086577a023d (HEAD -> main, branchA)
| Author: V Kumar <vkcode7@yahoo.com>
| Date:   Mon Mar 4 10:46:54 2024 -0500
| 
|     testing ff merge
| 
* commit 6bd689e65797f68ad3a9503772319e402fba667e (origin/main)
| Author: vkcode7 <vkcode7@yahoo.com>
| Date:   Thu Feb 29 22:33:30 2024 -0500
| 
|     Update README.md
| 
* commit 3b2e91bdb702702cce34fe52cae3a8dcaf2e1ede
| Author: vkcode7 <vkcode7@yahoo.com>
| Date:   Thu Feb 29 15:01:47 2024 -0500
| 
|     Update README.md
| 
* commit 1b99ffa32bb36587f98ab6f0fd12993b882a2eca
| Author: vkcode7 <vkcode7@yahoo.com>
```
- As you see, ReadMe.md is now behind and git push will fail with following message
```bash
(base) gs@GSs-MacBook-Pro gitdemo % git push
To https://github.com/vkcode7/github-demo.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/vkcode7/github-demo.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```bash

To resolve this, you need to do a "git pull" followed by "git push"

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git log --graph
*   commit 88e226cdd31d87396998dd66941a01b117c302b2 (HEAD -> main, origin/main)
|\  Merge: 05a574f 02a037d
| | Author: V Kumar <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 10:49:29 2024 -0500
| | 
| |     Merge branch 'main' of https://github.com/vkcode7/github-demo
| | 
| * commit 02a037dbd8ccdb17601fb37ac6d6d8b830e71e53
| | Author: vkcode7 <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 10:41:32 2024 -0500
| | 
| |     Update README.md
| |     
| |     testing FF merge
| | 
| * commit 96b6365ac62ed3901c8b21f715a312a8bb30e0a8
| | Author: vkcode7 <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 10:35:18 2024 -0500
| | 
| |     Update README.md
| | 
* | commit 05a574f68dc4205daa1a766b35047086577a023d (branchA)
|/  Author: V Kumar <vkcode7@yahoo.com>
|   Date:   Mon Mar 4 10:46:54 2024 -0500
|   
|       testing ff merge
| 
* commit 6bd689e65797f68ad3a9503772319e402fba667e
| Author: vkcode7 <vkcode7@yahoo.com>
| Date:   Thu Feb 29 22:33:30 2024 -0500
| 
|     Update README.md
| 
```

### Another merge strategy
- make a change in branchA to FileC.txt and commit
- switch to main and make a change in FileA.txt and commit
- merge branchA into main
  
```bash
(base) gs@GSs-MacBook-Pro gitdemo % git checkout branchA
Switched to branch 'branchA'
(base) gs@GSs-MacBook-Pro gitdemo % git commit -am "updated"
[branchA fa33f22] updated
 1 file changed, 2 insertions(+), 1 deletion(-)
(base) gs@GSs-MacBook-Pro gitdemo % git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
(base) gs@GSs-MacBook-Pro gitdemo % git commit -am "updated via main"
[main dec0d49] updated via main
 1 file changed, 2 insertions(+), 1 deletion(-)
(base) gs@GSs-MacBook-Pro gitdemo % git merge branchA
Merge made by the 'recursive' strategy.
 FileC.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
(base) gs@GSs-MacBook-Pro gitdemo % git log --graph
*   commit a31e045f574c2977717d5eea9553bf2712578e4c (HEAD -> main)
|\  Merge: dec0d49 fa33f22
| | Author: V Kumar <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 11:04:52 2024 -0500
| | 
| |     Merge branch 'branchA'
| | 
| * commit fa33f22b91d75bf7312d73833e0ce84a679f3c06 (branchA)
| | Author: V Kumar <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 11:04:04 2024 -0500
| | 
| |     updated
| | 
* | commit dec0d495642dd0d4940dd4b729933d59ee88627b
| | Author: V Kumar <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 11:04:44 2024 -0500
| | 
| |     updated via main
| |   
* |   commit 88e226cdd31d87396998dd66941a01b117c302b2 (origin/main)
|\ \  Merge: 05a574f 02a037d
| |/  Author: V Kumar <vkcode7@yahoo.com>
|/|   Date:   Mon Mar 4 10:49:29 2024 -0500
| |   
| |       Merge branch 'main' of https://github.com/vkcode7/github-demo
| | 
| * commit 02a037dbd8ccdb17601fb37ac6d6d8b830e71e53
| | Author: vkcode7 <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 10:41:32 2024 -0500
| | 
| |     Update README.md
| |     
| |     testing FF merge
| | 
| * commit 96b6365ac62ed3901c8b21f715a312a8bb30e0a8
| | Author: vkcode7 <vkcode7@yahoo.com>
```

After the above I updated ReadMe.md on github and pushed it. The log then looks like below:
```bash
(base) gs@GSs-MacBook-Pro gitdemo % git log --graph
*   commit bbb71c7dac135842669c0d83442200259238207d (HEAD -> main)
|\  Merge: a31e045 6b4da79
| | Author: V Kumar <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 11:08:13 2024 -0500
| | 
| |     Merge branch 'main' of https://github.com/vkcode7/github-demo
| | 
| * commit 6b4da790a62b07d736814ef76f52062a4ad3610a (origin/main)
| | Author: vkcode7 <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 11:08:05 2024 -0500
| | 
| |     Update README.md
| | 
| * commit d31c73bf75e563cf4d54a4ec34bbd441ee286737
| | Author: vkcode7 <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 10:57:38 2024 -0500
| | 
| |     Update README.md
| |   
* |   commit a31e045f574c2977717d5eea9553bf2712578e4c
|\ \  Merge: dec0d49 fa33f22
| | | Author: V Kumar <vkcode7@yahoo.com>
| | | Date:   Mon Mar 4 11:04:52 2024 -0500
| | | 
| | |     Merge branch 'branchA'
| | | 
| * | commit fa33f22b91d75bf7312d73833e0ce84a679f3c06 (branchA)
| | | Author: V Kumar <vkcode7@yahoo.com>
| | | Date:   Mon Mar 4 11:04:04 2024 -0500
| | | 
| | |     updated
| | | 
* | | commit dec0d495642dd0d4940dd4b729933d59ee88627b
| |/  Author: V Kumar <vkcode7@yahoo.com>
|/|   Date:   Mon Mar 4 11:04:44 2024 -0500
| |   
(base) gs@GSs-MacBook-Pro gitdemo % 
```
I will then have to do a "git pull" to bring ReadMe.md changes and then issue a "git push", to push my changes to remote main. The log will then looks like this:

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git push
Enumerating objects: 19, done.
Counting objects: 100% (15/15), done.
Delta compression using up to 12 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (10/10), 1.11 KiB | 1.11 MiB/s, done.
Total 10 (delta 3), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (3/3), completed with 1 local object.
To https://github.com/vkcode7/github-demo.git
   6b4da79..bbb71c7  main -> main
(base) gs@GSs-MacBook-Pro gitdemo % git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
(base) gs@GSs-MacBook-Pro gitdemo % git log --graph
*   commit bbb71c7dac135842669c0d83442200259238207d (HEAD -> main, origin/main)
|\  Merge: a31e045 6b4da79
| | Author: V Kumar <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 11:08:13 2024 -0500
| | 
| |     Merge branch 'main' of https://github.com/vkcode7/github-demo
| | 
| * commit 6b4da790a62b07d736814ef76f52062a4ad3610a
| | Author: vkcode7 <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 11:08:05 2024 -0500
| | 
| |     Update README.md
| | 
| * commit d31c73bf75e563cf4d54a4ec34bbd441ee286737
| | Author: vkcode7 <vkcode7@yahoo.com>
| | Date:   Mon Mar 4 10:57:38 2024 -0500
| | 
| |     Update README.md
| |   
* |   commit a31e045f574c2977717d5eea9553bf2712578e4c
|\ \  Merge: dec0d49 fa33f22
| | | Author: V Kumar <vkcode7@yahoo.com>
| | | Date:   Mon Mar 4 11:04:52 2024 -0500
| | | 
| | |     Merge branch 'branchA'
| | | 
| * | commit fa33f22b91d75bf7312d73833e0ce84a679f3c06 (branchA)
| | | Author: V Kumar <vkcode7@yahoo.com>
| | | Date:   Mon Mar 4 11:04:04 2024 -0500
| | | 
| | |     updated
| | | 
* | | commit dec0d495642dd0d4940dd4b729933d59ee88627b
| |/  Author: V Kumar <vkcode7@yahoo.com>
|/|   Date:   Mon Mar 4 11:04:44 2024 -0500
| |   
```

### Squash merge
Suppose a dev is working on many branches and therefore have a detailed log history due to multiple commits (no push yet). Now when it will be merged in main, all those messages will be in the main too. Suppose we dont want all that history in logs after the merge, for that we can use a squash merge. So it helps reduce the commit messages in git log.<br>
The command for squash merge is: git merge --squash branchA

### GitHub merge
On GitHub we can create a new branch, work on the changes and commit them. When we create a new branch, all the code from main gets copied over onto new branch.

After commit, we should then do a pull first by creatin a pull request, after that we can perform "merge pull request". While merging you have option to select a normal merge, squash or a rebase merge.

### git push -u origin -all
It pushes all changes from all branches to remote repo.

## Branch protection rule
Git allows various rules to be enforced on a branch (usually main) such as a pull request before merge, approvals for merge etc. Read more about "creating a pull request" to know more.

## Tagging
Can be used to specify important commits. Can also be used to mark release points. 2 types are their - lightweight and annotated.<br>
Leightweight tags are just a pointer to a specific commit.<br>
Annotated are stored as full objects in git repo. These are checksummed, contains the tagger name, email, date and message. This is how you create annotated tag on current commit.
git tag -a "v1.0" -m "this is version 1".<br>
"git tag" will display all the tags

```bash
(base) gs@GSs-MacBook-Pro gitdemo % git tag -a "v1.0" -m "this is version 1.0" 
(base) gs@GSs-MacBook-Pro gitdemo % git log
commit 3faaac464cbcf2b8d12fdc7b976ab14d6483be25 (HEAD -> main, tag: v1.0, origin/main)
Author: vkcode7 <vkcode7@yahoo.com>
Date:   Mon Mar 4 11:13:59 2024 -0500
```

"git show v1.0" will show details of the specific commit.

## Git maintenance
- git gc: kind of garbage collect, deletes non referenced objects
- git prune: to prune loose object

## Git LFS - Large File System
For large files such as image or vids, you can use GIT but for that first install Git LFS using "git lfs install". Git then uses LFS system to store those files and pointers in git repo so you use them normally via git repo.

## Cherry Picking
Suppose we have main branch master. We have FileB.txt in which we make 2 changes/commit via FeatureB branch.

```bash
(base) gs@GSs-MacBook-Pro azrepo % git branch
* master
(base) gs@GSs-MacBook-Pro azrepo % git checkout -b FeatureB
Switched to a new branch 'FeatureB'
(base) gs@GSs-MacBook-Pro azrepo % git commit -am "change 1"
[FeatureB 486a171] change 1
 1 file changed, 1 insertion(+)
(base) gs@GSs-MacBook-Pro azrepo % git commit -am "change 2"
[FeatureB da51027] change 2
 1 file changed, 2 insertions(+), 1 deletion(-)
(base) gs@GSs-MacBook-Pro azrepo % git log --graph
* commit da5102743317f5f56499c3f11de64a5b68c2d9a9 (HEAD -> FeatureB)
| Author: V Kumar <vkcode7@yahoo.com>
| Date:   Mon Mar 4 16:30:00 2024 -0500
| 
|     change 2
| 
* commit 486a17134031775a78c6b911ffb9144c8ec46fad
| Author: V Kumar <vkcode7@yahoo.com>
| Date:   Mon Mar 4 16:29:39 2024 -0500
| 
|     change 1
| 
* commit 74d45cb992b4d871dd6adae994bccfe6bbc1f8cc (origin/master, master)
| Author: V Kumar <vkcode7@yahoo.com>
| Date:   Mon Mar 4 12:59:40 2024 -0500
| 
|     another change
| 
* commit 9ad616c0b86ec716ee79345934342b26bd026fb2
  Author: V Kumar <vkcode7@yahoo.com>
  Date:   Mon Mar 4 12:53:19 2024 -0500
  
      initial commit
```

Now we have 2 changes in FileB.txt on FeatureB and these are not in main branch.

Suppose we want only change 1 in main branch we can do that via cherry picking:

```bash
(base) gs@GSs-MacBook-Pro azrepo % git checkout master 
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
(base) gs@GSs-MacBook-Pro azrepo % git cherry-pick 486a17134031775a78c6b911ffb9144c8ec46fad
[master cb576af] change 1
 Date: Mon Mar 4 16:29:39 2024 -0500
 1 file changed, 1 insertion(+)

(base) gs@GSs-MacBook-Pro azrepo % git log --graph
* commit cb576af11953070edd93afd8ec48909300ed55f0 (HEAD -> master)
| Author: V Kumar <vkcode7@yahoo.com>
| Date:   Mon Mar 4 16:29:39 2024 -0500
| 
|     change 1
| 
* commit 74d45cb992b4d871dd6adae994bccfe6bbc1f8cc (origin/master)
| Author: V Kumar <vkcode7@yahoo.com>
| Date:   Mon Mar 4 12:59:40 2024 -0500
| 
|     another change
| 
* commit 9ad616c0b86ec716ee79345934342b26bd026fb2
  Author: V Kumar <vkcode7@yahoo.com>
  Date:   Mon Mar 4 12:53:19 2024 -0500
  
      initial commit
```

## Good practices
- Always maintain a high quality for main branch (keep prod code in that)
- create branches for your features and bug fixes [git checkout -b branchA]
- commit the changes [git commit -am "message"]
- use push to push the changes to remote git repo (git push -u origin branchA) [branchA on remote repo will be updated]
- use pull requests to merge your feature branch into main [under branches, select branchA and then create a new pull request on web UI]
- when pull requests is created, reviewers can be added to review it or you can click "complete pull request" and sleect the merge type (merge (no FF), squash, rebase), and complete the merge process
- after the above merge process, the main will have changes from branchA (the branchA will be deleted if during merge process we select the option to delete it after merge)
- keep your feature branches short lived and delete it after merge
- you may need to merge first in release branch which is merged later to main

## Azure Repos

Goto Azure Repos under your Azure Project, from there generate credentials and copy the repo URL for use. Following shows how FileA.txt and FileB.txt were added to azure repo from local disk:

```bash
(base) gs@GSs-MacBook-Pro azrepo % git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint:   git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint:   git branch -m <name>
Initialized empty Git repository in /Users/gs/Documents/azure/azrepo/.git/
(base) gs@GSs-MacBook-Pro azrepo % git add .
(base) gs@GSs-MacBook-Pro azrepo % git commit -m "initial commit"
[master (root-commit) 9ad616c] initial commit
 2 files changed, 2 insertions(+)
 create mode 100644 FileA.txt
 create mode 100644 FileB.txt
(base) gs@GSs-MacBook-Pro azrepo % git remote add origin https://vkcode7@dev.azure.com/vkcode7/AgileProject/_git/AgileProject
(base) gs@GSs-MacBook-Pro azrepo % git push -u origin --all
fatal: Authentication failed for 'https://dev.azure.com/vkcode7/AgileProject/_git/AgileProject/'
(base) gs@GSs-MacBook-Pro azrepo % git push -u origin --all
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 269 bytes | 269.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Analyzing objects... (4/4) (3 ms)
remote: Validating commits... (1/1) done (0 ms)
remote: Storing packfile... done (89 ms)
remote: Storing index... done (46 ms)
To https://dev.azure.com/vkcode7/AgileProject/_git/AgileProject
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
(base) gs@GSs-MacBook-Pro azrepo % 
```

We can make changes to file, commit and push to azure repo. Under Azure Repo -> Files, we can see the changes and history

```bash
(base) gs@GSs-MacBook-Pro azrepo % git commit -am "another change"
[master 74d45cb] another change
 1 file changed, 2 insertions(+), 1 deletion(-)
(base) gs@GSs-MacBook-Pro azrepo % git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 295 bytes | 295.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Analyzing objects... (3/3) (3 ms)
remote: Validating commits... (1/1) done (0 ms)
remote: Storing packfile... done (53 ms)
remote: Storing index... done (34 ms)
To https://dev.azure.com/vkcode7/AgileProject/_git/AgileProject
   9ad616c..74d45cb  master -> master
```

Note: 
- the github repo can be imported in azure repo using github clone URL. Entire history will be imported too.
- we can also push a project to azure repo directly from visual studio for first time
- we can also setup branch policy in azure repo and force a review of pull requests.

### Pull requests and merge process
  - UserC create a new branch branchC, commit changes and create a pull request. The screen will show that atlease 1 reviewer must approve (forced via branch policy).<br>
  - A reviewer logs in, go to Azure Repos -> Pull Requests. It shows Mine, Active, Completed, Abandoned tabs. Under Active, pull request for branchC will show up<br>
  - Reviewer clicks on Approve dropdown button and can Approve, Approve with suggestions, Wait or Reject the pull request. Lets say he approved.
  - UserC now go to Repos -> Pull requests, he will see that it has been approved. He can now select "Complete dropdown", select Complete or Abandon. Click on Complete and select "Complete" which will ask for merge type to use and select it branchC should be deleted after merge. Select and click on "Complete merge". That will complete the merge process.
  
### Linked work items
Under "Branch policies", one can enable "Check for linked work items". This forces the pull request to be associated with Work Items (Tasks created for user stories on Azure Boards). You have to select the work items from drop down that are to be associated with the changes made in the pull requests.

## .gitignore
The .gitignore is a special type of file that can have file names or patterns of what not to track.

## Azure Boards and GitHub
One can install azure boards app by going to github market place (https://github.com/marketplace), search for Azure Boards and install there. You can then connect your Azure DevOps account to github.
Then the github repo can be connected to azure boards so that tasks can be linked with github pull requests and vice versa, the pull request will be linked to task in az boards. So complete tracking can be implemented between github and az boards. 

## Azure Repos and MS Teams/Slack
Azure Repos can be integrated with MS Teams/Slacks. With that anytime something happens in repo (new story created/updated, pull request etc), you will get a notification in MS Teams/Slack. Refer more on microsoft website on how to integrate Azure Repos with Slack or MS Teams.

## Checking/Disconnecting/Connecting remote origin

Verify remote origin. This command will list the existing remote repositories associated with your local repository.

```bash
git remote -v
```

Remove the current remote origin:
```bash
git remote remove origin
```

Add a new remote origin:
```bash
git remote add origin <new_remote_url>
```

```bash
git remote add origin https://vkcode7ad@dev.azure.com/vkcode7ad/BasicProject/_git/BasicProject
git push -u origin --all
```

The above will push the code from /Users/gs/Documents/azure/webapp/webapp to azure repo.

fatal: Authentication failed for 'https://dev.azure.com/vkcode7ad/BasicProject/_git/BasicProject/'
I resolved it by running following command and then entering the password on prompt and hitting enter

git credential approve <<< "protocol=https
host=https://dev.azure.com/vkcode7ad
username=vimalmalik
password=PASSWORD_KEY_FROM_REPO"

