# Basic git commands

1. [Identifying yourself to git](#1-identifying-yourself-to-git)
2. [Creating a git repo](#2-creating-a-git-repo)
3. [Files to Track and Initial Commit](#3-files-to-track-and-initial-commit)
4. [Recovering from Committed Mistakes](#4-recovering-from-committed-mistakes)
5. [Recovering from Uncommitted Mistakes](#5-recovering-from-uncommitted-mistakes)
6. [Branch Related Commands](#6-branch-related-commands)
7. [Github Push](#7-github-push)
8. [Miscellaneous Commands](#8-miscellaneous-commands)
9. [Version Controlling Terminology](#9-version-controlling-terminology)


### 1. Identifying yourself to git

```
$ git config --global user.name "Your Name"
$ git config --global user.mail "yourmail@example.com"
```

This is going to be the committer ID (author) in the output of "git log"


### 2. Creating a git repo

1. Change into working directory of your project which you are planning to do version controlling.
1. Initiate git.
    

```
$ cd /var/www/html
$ git init
```


### 3. Files to Track and Initial Commit

1. First, add the whole directory to be tracked.
2. Then remove the files or directory to be ignored.
3. Finally, enter the commit command.
```
$ git add .
$ git rm -r --cached ./img
$ git status  # shows the files to be ignored or to be staged
$ git commit  -m "Initial Commit"
```


### 4. Recovering from Committed Mistakes

For instance, change some file and stage (git add filename) it to be committed, then commit.

```
$ git add lines_deleted_some_code.py
$ git commit -m "deleted lines accidentally test"
```
1. You can see all the commits in the output of "git log" or "git log --oneline". We need the commit number that we want to revert the mistakes occurred in that cycle.
1. Then we can use that number to revert as in the second command below.

```
$ git log --oneline
ec6102e (HEAD) deleted a line accidentally test

$ git revert --no-edit ec6102e
```

After that, all mistakes will be gone and the output of "git log" should be something like that below.

```
$ git log --oneline
a611eb2 (HEAD) Revert "deleted a line accidentally test"
ec6102e deleted a line accidentally test
```


### 5. Recovering from Uncommitted Mistakes

To see changes since last commit till that moment:

`$ git diff HEAD`

And if there is a need to revoke all changes and revert to last commit:

`$ git reset HEAD --hard`


### 6. Branch Related Commands

###### _**1. See branches before clone(fork)**_
```
$ git branch
* master
```

###### _**2. To create a new_feature_branch from master repo and see the branches after the branch command:**_
```
$ git branch new_feature_branch

$ git branch
* master
  new_feature_branch
```

The preceding asterisk sign above shows that the master branch is active.

**_3. To activate the branch with name new_feature_branch:_**
```
$ git checkout new_feature_branch

Switched to branch 'new_feature_branch'

$ git branch
  master
* new_feature_branch
```

Since new_feature_branch is activated, any change made after that moment will not effect the master branch.

_4. After the changes made in that branch and if it is sure to update the master with these changes;
the commands to be used are:_

```
$ git checkout master
Switched to branch 'master'

$ git branch
* master
  new_feature_branch

$ git merge new_feature_branch
Updating e30cb27..e872c7b
Fast-forward
 test.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

_5. Now that master is updated and new_feature_branch is not needed; to delete this unnecessary branch:_

```
$ git branch -d new_feature_branch
Deleted branch new_feature_branch (was e872c7b).
```


### 7. Github Push

First, create a repository on your github account, for instance say it "*test*".
Then, on your terminal go to your git initiated project directory and issue these commands:

```
git remote add origin https://github.com/*your-github-username*/__test.git__
git push -u origin master
```
After the last command, you will be asked for your account credentials.


### 8. Miscellaneous Commands

- To see differences between commits; first, see the commit number in the output of:

```
$ git log --oneline
b8e791f (HEAD -> master) newline test
75d522c added some more
38726ed (origin/master) added some lines
d55f940 Initial Commit
```

Then note down the commit numbers which you want to compare. Differences between those commits are shown belows.
The output tells that the lines beginning with minus sign(-) are
removed and the ones with plus sign(+) are added:

```
$ git diff 38726ed b8e791f
diff --git a/git/git-notes-1.md b/git/git-notes-1.md
index 1c38d1d..0e9c4b0 100644
--- a/git/git-notes-1.md
+++ b/git/git-notes-1.md
@@ -127,4 +127,10 @@ Then, on your terminal go to your git initiated project directory and issue thes
 git remote add origin https://github.com/*your-github-username*/__test.git__
 git push -u origin master

-After the last command, you will be asked for your account credentials.
\ No newline at end of file
+After the last command, you will be asked for your account credentials.
+
+### 8. Miscellaneous Commands
+
+1. To see differences between commits first, see the commit number in the output of:
+
+$ git log --oneline
```

- What is changed since last commit:

```
git diff HEAD
```

- How to see branching with graph-like console output:

```
$ git log --graph --oneline --all
*   c374357 (HEAD -> master) fixed conflicts
|\  
| * d7f9e2c (new_doc_branch) a line is added in new_doc_branch for test
* | 53e8ea5 added some new items in master
|/  
* a916785 some new misc commands
* b8e791f newline test
* 75d522c added some more
* 38726ed (origin/master) added some lines
* d55f940 Initial Commit
```


### 9. Version Controlling Terminology

**Term** | **Description**
--- | ---
**revert/rollback** | _discard changes and return to one of previous commit_
**push/export** | _send changes from one repo (usually the local one) to another_
**pull/import** | _update (local) working set with remote repo_
**tag/label** | _name a specific state of a repository_
**branch/fork** | _make a clone of a repository_
**merge** | _integrate a branch (clone) back into the master repo (main branch)_

