# Introduction to Git by Stav Alfi

<img src="http://pvsousalima.github.io/grupython_apresentacao/images/Git-Icon-Black.png" width="70" height="70">

### Topics

1. [Introduction](#introduction)
2. [Repository structure](#1-repository-structure)
3. [What are Branches](#2-what-are-branches)
4. [Merging & Conflicts](#3--merging--conflicts)
5. [Rebase & Going deeper](#4-rebase--going-deeper)

###### Commnads:
1. [Areas](#areas)
2. [Branchs](#branchs)
3. [Merges and rebases](#merges-and-rebases)
4. [Remote](#remote)
5. [Stash](#stash)
6. [Logs](#logs)

The end of this article contains the [complete terms table](#complete-terms-table).
---

### Introduction

The tutorial is intended for programmers that want to dive deep and understand what is going on behind the scene. After reading and practicing you should be able to manipulate and understand more advanced articles online about other topics that are not mentioned here. 

You will learn familiar concepts and fully understand their definitions.

__Prerequirements__

No requirements.

### Tutorial


#### 1. Repository structure

Git is a source control system, and it builds on areas to have better control about what you are doing.



> **Definition 1.1.** _**Working space** is the main folder of your local repository. It contains tracked/untracked files/directories by git. 
> Files/directories that Git is tracking on will be called as tracked.
> Files/directories that Git is not tracking on will be called as untracked.
> Files/directories that Git is tracking on and those files/directories modified will be called as Modified but not staged_.

Git does not track on files or directories if you don't tell him to. To tell Git to start tracking on a file to look for future changes or to copy the changes you did to a specific tracked file to the stages area: 
`git add <file name A>`. Then all the changes will be added to the staged area to be committed later.
To commit: `git commit -m <reason for the commit>`.


> **Definition 1.2.** _**Index** is a file under the `.git` folder in your working directory that contains all the changed you did for tracked files/directories under the working directory. We also refer it as the **staged area**._

> **Definition 1.3.** _The working space is changing over time. A group of changes that you want to save and go back to in the future is called a **commit**. Commit is a file that contains changes in your project compared to the last change you saved and also contain an ID called **SHA-1**._

Commits are created from changes in the stage area.
They also contain the SHA-1 of the commit that created before them on the same branch, Who is the commit's creator and some other details. You can look over all the local commits by running `git log`.

![commits](https://i.stack.imgur.com/IY4PK.png)


> **Definition 1.4.** _**Local repository** refers to all the commits in your local project._

By committing some changes over time, you will create a graph that each node is a commit that looks at his _father_ commit. If you want to go back to a specific commit, Git will rebuild the repository history again but will stop at the commit that HEAD is pointing to.
(We will Define and talk more about what HEAD means later on).


#### 2. What are Branches

To let multiple developers work and change the same repositories concurrently and independently, we use branches.

> **Definition 1.5.** _**Branch** is the pointer (can and will be reassigned) to a specific commit. They contain the SHA-1 of the commit they are pointing to. `HEAD` usually is a pointer to the current branch you are working on, and we refer the branch that the `HEAD` is pointing to be the **working branch**_

When you are committing new commit, a new commit will be created on top of your branch. That means the working branch to be now pointing to the new commit (The new commit is pointing to the old commit, so no commits are lost by this process).
A branch stored in a file under the `.git\ref` folder. There are 2 types of branches: Locals and remotes (We will talk about remote branches later). The locals are located under the `heads` folder. 
When You create a new repository, you will be assigned the `master` as the main branch.
You can create multiple branches that will point to any commit you choose by running `git checkout -b <name of the new branch - A> <existing commit SHA-1 - B>`. To change the branch that `HEAD` is pointing to, run `git checkout <existing branch name - A>`.

                       branch master
                   /
    A1 <--A2 <--A3 <--A4 
                        \
                         branch testing <- HEAD


__Why do we need branches?__

For instance, when we will create a new commit under the testing branch, the testing branch will be pointing to the new commit but the master branch will not change.

                         branch master
                    /
    A1 <--A2 <--A3 <--A4 <--A5 
                              \
                               branch testing <- HEAD

Now changing `HEAD` to point to the master branch and commit again, the testing branch won't change but only the master branch.

                         A6 <- branch master <- HEAD
                    /
    A1 <--A2 <--A3 <--A4 <--A5 
                              \
                               branch testing
Now we have a nonlinear history. In other words, there is more than one commit that points to commit `f30ab`.

After we tested some code in the testing branch, we want to update our master branch with our changes (In case the master branch is the deployment branch, and the testing branch is the development branch).

#### 3.  Merging & Conflicts

    A1--A2--A3--A4 (branch development)
     \
      A11--A12 (branch master)

You created a branch for development and finished with some features. Currently, you are in the _master_ branch, and you want to update the _master_ branch with the commits: _A2, A3, A4_. By running `git merge development`, you are creating a new commit in the master branch _A13_ that contain the __changes that the _master_ branch doesn't have and the _development_ branch have__.

    A1--A2--A3--A4 (branch development)
     \          \
      A11--A12-----A13 (branch master)

Notice that the _master_ branch is now looking at commit _A13_ and commit _A13_ is pointing to both _A4_ and _A12_. That's a particular case in git where a commit is pointing to more than one commit.

The problem with merges that they make a _non-linear history_.

> **Definition 1.6** _**Non-linear history**_ is a history that contains a commit that points to more than one commit. It makes the history log difficult to look at and manage it. Try to avoid it as much as you can.
> **Definition 1.7** _**merge commit**_ is a commit created by a merge and usually points to multiple commits.
> 
__Conflicts__

Sometimes the merge operation doesn't go so well because:
1. Both commits (_A4_ and _A13_) updated or created the same file __after__ the closet ancestor commit _A1_.
2. (?)

In such cases, Git will let you solve the conflicts by showing you both file versions in that file and let you decide which version you want. When you finish, stage your changes and then commit. 


#### 4. Rebase & Going deeper

An alternative in most cases of merging will be `rebase`. It will recreate the history from a specific point to a specific point.
We had the following history:

    A1--A2--A3--A4 (branch development)
     \
      A11--A12 (branch master)

We want to add the commits: _A2, A3, A4_ to branch _master_. First, go to _development_ brach, the branch you want to rebase and run `git rebase master`- That will recreate commits _A2, A3, A4_ in master branch and set branch development to point the copy of commit _A4_.

```
    A1     (branch master)
      \      /
      A11--A12--A2`--A3`--A4` (branch development) 
```

__What does rebase do__

Attempting to rebase branch _development_ on brach _master_ will start copy all the commits needed one by one - From the child of the closest ancestor of both branches to the top commit in branch _development_. The copying of each commit is _Cherry-pick_. When the `rebase` is starting, you are entered to _detached head_ state, and each commit __will try to__ cherry-pick on top of branch _master_.

> **Definition 1.8** _**Detached head state**_ occurs when the _HEAD_ is not pointing to a local branch. Even if you are pointing to a remote branch, you are still in that state. Each new commit will be on top of that commit _HEAD_ is pointing to. Not any of your branches will point to the new commits you did in _detached head state_. Git will give those new commits 30 days until they will be removed automatically. 

In case of a conflict happened by the cherry-pick, you need to solve it and then run `git rebase --continue` or in case you want to stop the rebase, run `git rebase --abort`. By the time each cherry-pick ends, the new commit is identical to the old commit but with a new SAH-1.

Caution: Rebase does not know how to deal with merge commits.
When using rebase on a branch that had a _merge commit_, Git will ignore that commit and _cherry-picking_ each path that the merge commit is pointing to. The order of which path it will _cherry-picking_ is random unless you specify it your self.

______


### Most Asked Questions

__Q:__ My push failed because someone pushed before I did. Why did it fail?

__A:__ The push is not _fast forward_.  Consider the following case where you try to push to a branch with the same name as the branch you are working on right now, `B`. `B` point to commit `2` and `origin/B` points to commit `1` (You didn't push your branch yet, so the server's branch doesn't have the up to date commits).
Also, commit `2` point to commit `1`:

```
    2 (B)
      \ 
         1 (origin/B) 
```
Now your teammate push (before you pushed) to branch `B` but you still didn't `fetch` so you don't know it yet. The __server__ graph is:

```
         3 (B)
        /
         1
```

Now you are trying to push but fail because your push is not _fast forward_. Why?  First, we should know that if Git allowed us to push in this situation, the result could be:

```
    2 (B)  3 (B is not pointing to this commit anymore!! BAD)
       \ /
          1
```

Secondly, _fast forward_ mean we could do merge/push /pull without losing commits. In other words, There must be a link from the first commit we are trying to push to the commit which the branch in the server is pointing to. As we see in this example, there is no such link. Linking means we can go from commit `2` to commit `3`. (Invalid link is going from commit `1` to commit `2` or `3` or going from commit `2` to commit `3`).

The solution is to `fetch` on our local machine. The following graph show to result after `git fetch`:

```
    2 (B - HEAD)  3 (origin/B)
       \ /
          1
```
Then we must merge or rebase. The following graph show to result after `git merge origin/B`: 

```
       4 (B - HEAD)
      /     \ 
     2    3 (origin/B)
      \  /
          1
```
Then our push will be _fast forward_ unless someone pushed before us again.

---

__Q:__ I’m in detached head state. What do I do?

__A:__ It is highly recommended that you create a new branch where you are unless you are only watching the working directory at this commit. The definition for _Detached head state_ :

> **Definition 1.7** _**Detached head state**_ occurs when the _HEAD_ is not pointing to a local branch. Even if you are pointing to a remote branch, you are still in that state. Each new commit will be on top of that commit _HEAD_ is pointing to. Not any of your branches will point to the new commits you did in _detached head state_. Git will give those new commits 30 days until they will be removed automatically. 

---

__Q:__ How do I edit a friend’s code written in a specific time in history?

__A:__ Tell him to push his branch and then you `fetch` so you can see his most up to date work at his branch. You can't `checkout` his branch without creating a new branch and **then commit new changes** because if git approved it, his branch in the server would point to new commit without you doing any explicit `push`. 



Comming soon: Everything about remotes.

---

### Most used commands
__Switching between areas, changing your working directory files and branches__

#### Areas
* _Untracked_ , _Modified but not staged_ (_Working directory_)
* _Staged_ (_Index_)
* _Commited_ (_local repository_) 
* _Online repository_

![areas in git](http://interactionprototyping.github.io/exercises/images/github/git-stages.svg)
[Picture source: interactionprototyping.github.io](http://interactionprototyping.github.io/exercises/)

1. `git add <existing file name - A>` - file A switch from _untracked_ to _staged_ or from _modified but not staged_ to _staged_. It depends where file A was before running the command.
2. `git commit -m <reason for why you committed (as a text)>` - Create new commit from the files in the _staged_ area. In case you need more than one word in the text, use "..your message..".
3. `git commit -am <reason for why you committed (as a text)>` - Create new commit from the files in the _Modified but not staged_ and _staged_ areas. In case you need more than one word in the text, use "..your message..".
4. `git reset HEAD~` - Remove your last commit and delete everything from the _stage_.
5. `git reset -- soft HEAD~` - Remove your last commit and put it in the _stage_ and does delete the current files on _stage_.
6. `git reset` - moving everything from the _stage_ to _modified but not staged_. If a file was _untracked_ before it was inside the _stage_, then it will be _untracked_ again.
7. `git commit --amend` - Remove your last commit and moving the last commit to _stage_ and doesn't delete the current files on _stage_.
8. `git reset HEAD <exising file name - A>` - move file A from _stage_ to the area it was before (tracked and committed by git or not tracked by git).
9. `git checkout -- <existing file name - A>` - If file A existed in the last commit, then file A will be now in the working directory as it was in the last commit. If not, the file will be as it was in the _stage_ phase.

#### Branchs

1. `git checkout <existing branch name - A>` - Git changes your working directory to be as it was in commit with SHA-1 that equals the value of branch A. 

Then the value of `HEAD` changes to the name of branch A (which is "`A`"). Also, if `A` is the name of the branch in the remote server (for example _origin/A_), and you want to see him (or add commits to him), this command creates a new branch, called, `A` (if it doesn't exist already) and now `HEAD` is pointing to `A`.
2. `git branch -d <existing branch name - A>` - Remove branch A from your local repository. No changes occur to your working directory.
3. `git branch <name of the new branch - A>` - Create a new branch with the name A in your local repository with the value of the last commit's SHA-1 ("ID") you committed.
4. `git checkout -b <name of the new branch - A> <existing commit SHA-1 - B>` - Create a new branch in your local repository with the name A with the value equals to B. Then change HEAD's value to A.
5. `git checkout -b <name of the new branch - A> <remote server short name - B>:<existing branch name in this server - C>` - Creating a new branch in your local repository with the name `A` that has the same value as branch `C`. 

Then `HEAD` is referencing now to `A` and your working directory will be changed to the commit that branch A points to.
6. `git checkout --track <remote server short name - B>:<existing branch name in this server - C>`- shortcut of the last command but now the new branch will be called `C`.
7. `git checkout <existing commit SHA-1 -A>` - change your working directory to look as it was in commit `A`. Change `HEAD`'s value to A's SHA-1. (You are now _Deteched head_ state and also `HEAD`'s value is an SHA-1 and not a reference to existing branch).

#### Merges and rebases

1. `git merge <existing branch name - A>` - Create a new commit _B_ in the top of the working branch (where _HEAD_ is pointing to). `B` points to both the last commit _HEAD_ pointed too, and the top commit in branch _A_. Commit _B_ contains the changes in both commits he points to. This command creates a _non linear_ history. Try to use `rebase` command as an alternative.
2. `git rebase <name of existing branch A>`- create a commit _b1_ to be at the top of the _working branch_, commit _a1_ to be the top of branch _A_ and commit _c1_ to be the closest ancestor of both _b1_ and _a1_. Copy all the commits from _b1_ to the child of _c1_ in the _working branch_ to the top of branch _A_. Rebasing does not know how to deal with _non-linear history_ if you don't specify how. In case you are rebasing a merge commit (a commit that points to multiple commits), then the merge-commit will not be copied during the `rebase` process. It means that Git will remove the merge-commit. 

The path order that Git will copy will be random. See more details in the tutorial.

#### Remote

1. `git clone <remote repository URL - A>` - Creating a local directory with the repository files and directories. Also does `git remote add shortRemoteRepositoryNameB A` (Command 2). Creating a local branch name master that points to the commit where shortRemoteRepositoryNameB/HEAD is pointing (The HEAD in the server).
2. `git remote add <remote repository short name - A> <remote repository URL - B>` - A will be a shortcut to B.
3. `git fetch <remote repository name - A>` - Update all your local repository to have the most up-to-date commits/branchs/.. from A. It does not change anything in your working directory and __does not__ create local branches with the same name as the remote branches and this command will not change your local HEAD.
4. `git push <remote repository short name - A> <local branch name - B>` - Asking the remote repository to add to remote branch B your commits in local branch B that it doesn't have. Conditions: (a) Changing the value of remote branch B should be _fast forward_. (b) Your repository must have a copy of the most up to date remote branch B.
5. `git pull` - Same as `git fetch` and `git merge @{u}`. The latter means merging __only__ your working branch with the remote branch that has the same name. If _fast forward_ occurred, then your local branch is not pointing to the same commit as the remote branch with the same name. If not, your local branch that involved with the merge will point to the new merge commit and _HEAD_ will. __It is recommended to use `git pull` to do only _fast forward___.
6. `git push --delete <remote server short name> <remote branch name- B>` - Deleting branch B the remote server but not local branch B (if it exists).

#### Stash

1. `git stash` - Its goal to save the work that we did not commit. In more details, Git will save the index content in a particular stack with the changed made to known files by git. This command will create two commits: 
* The first contains the content of the index file.
* The second contains the content of the changes we made to all of the files known by Git. 

The second commit points to the first commit and the first commit points to the commit where the active branch is pointing to. After that, Git will remove any changes we did to all the known files by Git since the last commit and also delete the content of the index file. 

For each additional stash, the saved data will be pushed to the stack so the older stashes' indexes will increase by one. Also, the commits created by the last stash will be removed, and new ones will be created with the same logic.

2. `git stash list` - show list of all stash exist in our stack. the newest stash is `0` and the oldest stash have the biggest number - equal to the number of active/saved stashes we made `-1`.
3. `git stash clean` - empty the stash stack.
4. `git stash pop` - try to pop out the last stash was saved in the stack by adding the changes from the index stash commit to our index and the changes from the second commit to our working directory. There may be conflicts need to be taken care of.
5. `git stash pop stash@{i}` - try to pop the stash saved in the stack with index `i`. If there are no conflicts, remove the poped stash from the stack.
6. `git stash apply stash@{i}` - `try to pop the stash saved in the stack with index.



#### Logs

8. `git diff <exsiting file name - A>` - Print the changes of file A in the _tracked but modified_ comparing to file A in the last commit.
9. `git diff -- cached <exsiting file name - A>` - Print the changes of file A in the _staged_ comparing to file A in the last commit.
10. `git diff <commit SHA-1 - A> <commit SHA-1 - B>` - print the differences between A and B.
11. `git remote -v` - print the short names of the remote repositories' URLs.

With Git, you can easily edit the log output as you wish.
There are some ready-made logs created by other. Go to your home directory and edit the file `.gitconfig`. Add the following lines to the end of the file.:

    [alias]
        lg = !"git lg1-specific --all"
        lg1-specific = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(auto)%d%C(reset)'

Now by running: `git lg` = `git lg1`, `git lg2`, `git lg3` you can see all your branches and commits throughout a graph.

---
### Complete terms table

> **Definition 1.1.** _**Working space**_ is the main folder of your local repository. It contains tracked/untracked files/directories by git. 
> Files/directories that Git is tracking on will be called as tracked.
> Files/directories that Git is not tracking on will be called as untracked.
> Files/directories that Git is tracking on and those files/directories modified will be called as Modified but not staged.

> **Definition 1.2.** _**Index**_ is a file under the `.git` folder in your working directory that contains all the changed you did for tracked files/directories under the working directory. We also refer to it as the _**staged area**_.

> **Definition 1.3.** The working space is changing over time. Each important change that you want to save and go back to in the future called a **commit**. Commit is a file that contains changes in your project compared to the last change you saved and also contain an ID called _**SHA-1**_.

> **Definition 1.4.** _**Local repository**_ refers to all the commits in your local project.

> **Definition 1.5.** _**Branch**_ is the pointer (can and will be reassigned) to a specific commit. They contain the SHA-1 of the commit they are pointing to. `HEAD` usually is a pointer to the current branch you are working on, and we refer the branch that the `HEAD` is pointing to be the **working branch**

> **Definition 1.6** _**Non-linear history**_ is a history that contains a commit that points to more than one commit. It makes the history log challenging to look at and manage it. Try to avoid it as much as you can.

> **Definition 1.7** _**merge commit**_ is a commit created by a merge and usually points to multiple commits.

> **Definition 1.8** _**Detached head state**_ occurs when the _HEAD_ is not pointing to a local branch. Even if you are pointing to a remote branch, you are still in that state. Each new commit will be on top of that commit _HEAD_ is pointing to. Not any of your branches will point to the new commits you did in _detached head state_. Git will give those new commits 30 days until they will be removed automatically. 

------------------

### Legal

© Stav Alfi, 2017. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear
credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Introduction to Git by Stav Alfi" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/1020abe20960c2daf215410da56250eb.