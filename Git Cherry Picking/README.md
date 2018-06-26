### Git Cherry Picking by Stav Alfi

<img src="http://pvsousalima.github.io/grupython_apresentacao/images/Git-Icon-Black.png" width="70" height="70">

### Topics

1. [Introduction](#introduction)
2. [What is a cherry-pick](#what-is-a-cherry-pick)
3. [How does it work](#how-does-it-work)
4. [Multiple commits at once](#multiple-commits-at-once)
5. [Quiz](#quiz)
6. [Legal](#legal)

---

### Introduction

The goal of this tutorial is for explaining what cherry-pick command is, how does it work and how to handle errors (conflicts).

This tutorial does not focus on real-life examples which involve cherry-picking because those examples may be objective. However, I do say that `git cherry-pick` is a __prerequirement__ to understand `git rebase` which will be covered in the future tutorial.

In this toturial I will explain the following cammands and files:
1. `git cherry-pick`
2. `cherry-pick --continue`
3. `cherry-pick --abort`
4. `git cherry-pick <commit1_hash> <commit2_hash> <commit3_hash> and so on`
5. `git cherry-pick <commit1_hash>..<commit2_hash>`
6. `CHERRY_PICK_HEAD` file

###### Prerequirements

I assume you fully understand the following git commands concepts:
1. `git am`.
2. `git diff` (especially _chunk_ and _chunk context_).
3. `git apply`.
4. `git merge-file`.
5. `git format-patch`.
6. _2 way merge_.
7. _3 way merge_.
8. _blob_.

Also if you are not familiar with Git source control, basic git commands can be found here: [Introduction to Git by Stav Alfi](https://gist.github.com/stavalfi/1020abe20960c2daf215410da56250eb)

---

### What is a cherry-pick

The [formal definition](https://git-scm.com/docs/git-cherry-pick) is:

> ___git-cherry-pick___ - Apply the changes introduced by some existing commits.

A quick definition to their definition can be: _cherry-pick_ means to copy a commit from somewhere and _git am_ it where the active branch is on.

For example,  in the following graph where`HEAD` is looking at the blue (top) branch, we want to copy commit 4 to the blue branch:

![](https://i.imgur.com/FGyDQKO.jpg)

```git
git checkout blue-branch
git cherry-pick commit-4
```
![](https://i.imgur.com/uthHw1m.jpg)

Theoretically, we could copy `commit-2` instead (technically, this will probably won't work; We will talk about it later):

![](https://i.imgur.com/p1ZPzNK.jpg)

### How does it work

When we are cherry-picking, Git does not copy a commit but check what has changed between the commit we want to copy, and it's parent. Then Git _try_ to create a new commit from that _patch_ on where the active branch is.  

I will describe the algorithm `git cherry-pick` use as pseudocode:

![](https://i.imgur.com/FGyDQKO.jpg)

```git
git checkout blue-branch
git cherry-pick commit-4
```

_Start of pseudo code_

1. Git uses `diff` between `commit-3` and `commit-4` and saves it as a `patch`:
`git format-patch -1 commit-4` - this means, create a patch from the diff between `commit-4` and `1` commit back which is `commit-3`.
2. Git tries to `apply` the patch we created on where the active branch is:
`git apply --index the-patch-git-created-in-step-1`. `--index` will apply the patch to the index also.
3. Success. Git commits all the changes in the index file to create a new commit. The active branch will point to the new commit. __Finish__.
4. Fail. Git fails to apply some of the chunks in the patch.  There can be 2 reasons for the failure:
      * if the _chunk_ is, for example, about a change occurred in file `A.txt`, Git didn't find file `A.txt` inside `commit-6`.
      * Inside the file which the chunk needs to be applied, Git didn't find the _chunk contex_ or Git found the context but the line it should change is missing or changed already. In other words, if the _chunk_ is, for example, about a change occurred in file `A.txt`, Git search inside file `A.txt` inside `commit-6` the _chunk context_ (3 lines before the lines which should be changed` or 3 lines after them), but it failed to find them or the lines which should be changed because those lines are missing or changes already.
5. The chunks which successfully applied will stay inside the working directory, and the index but the Git will try to do one last and single.. rescue for you with _3 way merge_ to the specific chunks which failed to apply.  Let us look inside a patch and analyze it:

```java
From 3282a02b0f3f1c49bcd3869a31c3842eaa76f7c6 Mon Sep 17 00:00:00 2001
From: stav alfi <stav.alfi@nice.com>
Date: Wed, 8 Nov 2017 22:11:40 +0200
Subject: [PATCH] 2

---
 1.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

diff --git a/1.txt b/1.txt
index 5bc6c8d..5affb52 100644
--- a/1.txt
+++ b/1.txt
@@ -17,5 +17,5 @@
 16
 17
 18
-19
+aa
 20
-- 
2.9.2.windows.1
```
If this was the patch which Git created after running `git cherry-pick commit-4`and failed to apply the only chunk here, then Git will try to do _3 way merge_ by specify the base to be the `blob` `5bc6c8d` (left side - the version of `commit-2`), and merge between `blob` `5affb52` (right side - the version of `commit-3`) and the `blob` of our version of this file (`1.txt` in the patch from the example above). 

If Git fails during the _3 way merge_, then it means Git found conflicts, and he will do 2 things:
* set the file `CHERRY_PICK_HEAD ` to point to the problematic commit we can't cherry-pick, which is `commit 4`. Why? I will answer this after the pseudo code.
* __You__ need to resolve those conflicts. Good luck. How? The same way we resolve _merge conflicts_ while running `git merge`. After that, run `git cherry-pick --continue` because maybe there are more conflicts (Git let you resolve at most 3 conflicts at once). If everything goes well from here and no conflicts are found, then a new commit will be created: `commit-4` in the active branch. We are done!

_End of pseudo code_

### Multiple commits at once

What if we want to cherry-pick multiple commits? For example: `git cherry-pick commit-2 commit-3 commit-4` ? Git does the same process over and over again from the most left commit to the most right commit in the command.
The exciting part is if there were conflicts while applying `commit 3`, how Git will know where to go back after you resolve the conflicts? Well, that's easy, Git saved a reference to the problematic commit (`commit-2` in this example) inside the `CHERRY_PICK_HEAD` file. So after resolving the conflicts, Git know which it commits cherry-pick now and will move to the next commit in the command above (`commit-3` in this example).

Same as if we want to: `git cherry-pick commit-1..commit-4`which means: cherry pick every commit from `commit 4` (`commit4` will be cherry-picked) until `commit 1`  (`commit1` will __not__ be cherry picked). In other words: everything from `commit-4` to `commit-2`.

### Quiz

1. can I cherry-pick a commit in my branch? There can be 2 reasons this can fail -go to step 4 in the pseudo code.

-----

### Legal

© Stav Alfi, 2017. Unauthorized use and/or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear
credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Git Cherry Picking by Stav Alfi" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/70ff71b3693a0eda6994550b91627af1.