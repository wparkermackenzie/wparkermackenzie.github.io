---
layout: default
title: "Git Reference Guide"
---

In 2012 I jumped into Git with both feet and eyes closed as the company I was working for transitioned away from CVS and Clear Case. The company was large enough to have a small team of individuals responsible for the maintenance of the repository, allowing me to be a contributor. At the time I found the concept of distributed repositories and the ability to revision all sources instead of individual files intriguing. However, as with all things, when you have the power of a star ship at your finger tips one can easily get lost in the sea of buttons, unable to remember how to disengage the locking clamps. Since that time I have been a contributor, an integration manager, and responsible for the repository with a centralized work flow. In that time, I have compiled the following notes to refer back to; these will have to suffice until I can ask Alexa "grab the changes that provide low power mode from the development branch". 

These Notes were last updated 2017-OCT-17

# References
* ["Pro Git"; Chacon, Scott](https://git-scm.com/book/en/v2)
* [Bit Bucket Tutorial](https://www.atlassian.com/git/tutorials/learn-git-with-bitbucket-cloud)

# Definition of Terms

* HEAD
    * A special branch name which represents a pointer to the branch one is currently on. This IS different than CVS.

* master
    * The default branch name. As commits are made the master moves to a branch of the last commit made.

* origin
    * The server from which repository changes will be cloned, pushed, and pulled from. When a repository is cloned, the command automatically adds the remote repository to the name origin.

## Typical Branch Naming Conventions
```
Master      A-----------------------E----H-------------->
             \                     /           
Development   B<--------C<--------D<-------E<----->
               \       /
Topic           F<----G
```

Master branch       : Stable code base only bug fixes of a high severity and
                      high priority are committed directly to 
                      the branch. Development branch may be merged in when it
                      has become stable.

Development branch  : This is the integration branch. As each of the topic
                      branches pass a unit test which verified the 
                      functionality it gets integrated into the development
                      branch for integration testing.

Topic/Feature Branch: Bug fixes and feature development work get their own
                      branch. Only when the feature is complete and passes
                      a unit test is it integrated into the development branch.

# Notation
* Commit ranges 
    * Double-dot notation
        * Answer the questions what is on this branch but not on that branch
        * X..Y : Commits that are in Y but not in X
        * X..  : Commits that are in HEAD but not X (short for X..HEAD)
    * Tripple-dot notation
        * Answer the questions what IS in this branch AND that branch but not the branches common references.
        * X...Y

* References to remote branches
    * (remoteName)/(remoteBranchName)
        * e.g. origin/master
            * Specifies the master branch on the origin remote

* SHA-1 and Short SHA-1 Notation
  * Each commit is associated with a 20 Byte SHA-1 hash 
  * This SHA-1 hash is used to specify a specific commit
  * When specifying the SHA-1 hash key for a commit only the first 8 
    characters are usually necesary. This is known as Short SHA-1 Notation

# Commands When and Why to Use Them

## git add
Add the files that need to be "staged" into the local repository. This not only adds new files to the repository but also must be done for files which have changed and will eventually be committed to the  repository (this is called staging).
```shell
git add xtitle
```
Note that if one does the following
1. git add file.txt
2. modify file.txt
3. git commit file.txt

**ONLY THE CHANGES FROM (1) WILL BE COMMITTED IN (3).** This allows one to stage changes without committing them...yet...
  
To unstage a file before it is committed 
This will remove file(s) from the staging area which were previously added but not committed.
Note: The git status command supplies the correct command to execute
```shell
git reset HEAD <file>
```

* Caution should be taken to specify the file to be added. If git add is run with no filename then all modified files and new files will be added to the staging area.

* If one modifies a file after it has been staged by running git add, one will need to run git add again to stage the latest version of the file.  Otherwise, on the version of the file associated with the initial stage will be committed.

* See "git commit -a" as a way of skipping the addition to the staging area for files which already exist.


## git bisect
If one knows that some piece of functionality is broken as of a certain release and know that the functionality worked in another release then git bisect can help narrow down the commits in the release. 

For this command there really isn't a way to put it succinctly, I always reference [Pro Git, Git Tools : Bisect Debugging](https://git-scm.com/book/en/v2/Git-Tools-Debugging-with-Git) for this one...


## git blame 
When you see an obvious bug and you want to tease a coworker for such an obvious mistake, but first you want to prove to yourself it was not you...

To produce an annotated file which indicates when the line was changed, why (SHA-1), and by whom:
```shell
git blame <filename>
```


## git branch
See the list of current local branches. In the listing the branch name with the * character indicates the HEAD (i.e. the current branch checked out).
```shell
git branch
``` 
List all branches
```shell
git branch -a
```

Create a new branch 
```shell
# At the same commit point presently on:
git branch <branchName>

# Branch from existing reference (tag or branch)
git branch <new-branch-name> <existing-ref>
```

Delete a branch that is no longer needed:
* If the branch is local This will fail if branchName contains work that is not merged in yet. To override this use -D; this will delete the branch and all the associated work.
  ```shell
    git branch -d <branchName>
  ```
        
* If the branch is remote _see git push_

List which branches have been merged to a given branch
```shell
git branch -a --merged <branchName>

# This is the same thing as 
git checkout master
git branch -a --merged
```

Change the name of a local only branch 
```shell
# While on the branch
git branch -m <newName>

# On a different branch
git branch -m <oldname> <newname>
```


## git checkout
Switch the local repository to an existing branch. This points HEAD to another the branch associated with branchName. Any changes and commits will now be associated with the branch switched to. It also reverts the files in the working directory to the snapshot taken at branchName.
```shell
git checkout <branchName>
```

Revert a modified file back to what it looked like when it last committed. **Danger the file is overwritten with the last version which was committed and this can not be undone.** 
```shell
git checkout -- <file>
```

Use the -b flag to create a new branch before switching to it. This is the same as using the branch option followed by another git command with the checkout option.
```shell
git checkout -b <newBranchName>
```

Create a tracking branch. There is a branch on the remote server which was brought down in the last fetch and one wants to work on that branch. GIT does not automatically create local editible copies of remote branches. To do this a tracking branch needs to be created then have the remote branch merged into. A   tracking is local and has a direct relationship with the remote branch.
```shell
git checkout -b <localBranchName> [remoteName]/[remoteBranchName]
# A simplified version which does the same thing:
git checkout --track## remoteName]/## remoteBranchName]
```

Copy a file from another branch into the current branch. This command will copy fileName (must be pathed if in a subdirectory) from branchName into the current branch. If branchName does not have a tracking branch then the full branch name (e.g. origin/branchName) must be used.
```shell
git checkout branchName fileName
```
	

## git cherry-pick
If you every fixed a bug on a development or feature branch and find that the fix is needed in a hot fix branch, this command is for you. Applies changes introduced in another commit into the currently checked out sources. Useful for taking a bug fix put into the trunk and back porting it into a bug fix branch.
```shell
git cherry-pick -x <commit>…
```

* The -x places a message into the log that the change was cherry picked and where it was picked from. 
  * e.g. git cherry-pick -x 7153cca
    * Places the change associated with hash 7153cca into the current sources. The hash was previously determined using git log

See the section Tasks below for more information


## git clone
Obtain a copy of the repository to be worked on

```shell
git clone <repository> <directory>
```
* Creates a working directory named <directory>
* Initializes the .git directory inside of this working directory, a local repository.
* Checks out a working copy of the latest sources into <directory>


## git commit
* To commit the changes to the local repository; brings up favorite editor. Put in useful comments and close the editor.:
```shell
git commit
```

* To specify the commit message on the command line use the -m flag:
```shell
git commit -m "Initial sources into the repository"
```

* To see exactly what changed, use the -v flag and git puts a diff in the comments of the commit message.
```shell
git commit -v
```

* To add files to the staging area and commit them in one command. Will only add and commit files which are already part of the repository.
```shell
git commit -a
```

* To amend/fix the last commit (add a file which was forgotten or to modify     the commit message) the amend option may be used. However, it must be run immediately after the previous commit as it uses the staging area. **Don't run this command after pushing the commit**
```shell
git commit --amend
```


## git config
Initial user configuration

The following sets the Git individual user's preferences and identity. The --global makes Git store this information in the user's .gitconfig file located in their home directory. Otherwise the information would be stored on a single machine or in a single repository.
```shell
git config --global user.name "W Parker Mackenzie"
git config --global user.email "wparkermackenzie@live.com"
git config --global core.editor /usr/bin/vim
git config --global merge.tool /usr/bin/gvimdiff
git config --global diff.tool p4merge
git config --list
```

## git diff
Viewing file changes : git diff

* To view what has been changed but not yet staged. Compares what is in working directory with what is in the staging area. Does NOT show all changes since last commit.
```shell
git diff
```
* To view what has been staged and will go into the next commit :
```shell
git diff --staged
```
* To view a list of files which have changed between branches and their status (commit may be a double-dot commit range):
```shell
git diff --name-status <commit>
```

* To view differences of a file between two different commit tags/branch-names/hashes
```shell
git diff <commit1> <commit2> fileName
```


## git fetch
Get data from a remote repository without merging that data into the local repository. :
```shell
git fetch <remote-name>
```
Gets all the data from the remote repository from which the local repository was initially cloned.
```shell
git fetch origin
```
                                                            

## git grep
Look for a pattern in the tracked files in the working tree
```shell
git grep <pattern>
```
* Usefull options
  * -n : display line numbers
  * -i : ignore case
  * -w : match on a word boundary
  * -c : Count the number of lines that matched


## git init
Create the initial repository

This will create what is known as a "bare repository". This is a repository which does not have any working files or directory structure; it is just the Git data.
* The following creates a new repository named reference
```shell
git init --bare reference.git
```
* The following makes the current working directory (myProj) a repository
```shell
cd /path/myProj
git init
git add .
```


## gitk
* To see a graphical commit history which is far more visual than git log the
  Tcl/Tk program distributed with Git may be used:
```shell
gitk
```

* The commit history is on the top half of the window followed by an ancestry graph. The bottom half has a diff viewer which shows the changed introduced at any selected commit.

Favorite options:
* Git-Flow like presentation. Provides a simplified view of the various tags and releases:
```shell
gitk --all --simplify-by-decoration &
# To see everything use the --all option
gitk --all &
```


## git log
* To see the commit history of one's commits or of the whole repository one
  uses Git's log command. To list the commits made in reverse chronological
  order :
```shell
git log
```

* My favorite which includes tag information:
```shell
git log --decorate 
```

* To include the differences introduced with each commit:
```shell
git log -p
```

* To limit the number of commits shown in the output use the -<n> option; this shows only the last 3 entries committed to the repository.:
```shell
git log -3
```
* To gather log statistics on multiple files use the --stat option. This prints below each commit entry the list of modified files, how many files were changed, and how many lines in the files were added and removed; all with a summary at the end:
```shell
git log --stat
```

To change the log output to formats other than the default the --pretty option is used.

* Prints the commits on a single line
```shell    
git log --pretty=oneline
```

* Prints the commits in ones own log output format. Useful for machine parsing. See the documentation in "Pro Git" location 449 for the tokenized values or the PRETTY FORMATS section of git log --help.
```shell
git log --pretty=format:"<tokenized string>"
```

* Get a log of all changes on a local branch (branch1) which were not merged
  onto another branch (branch2) switch to branch1 (e.g. git checkout branchName1) then use the following
```shell
git log branchName2..
```

* Obtain differences in the branch which are not on the master:
```shell
git log master..
```

* Create simple release notes between 2 versions:
```shell
git log --pretty=oneline  d5da1b318897...9284eeba91cb
```
 

## git merge
Merge a branch into the current branch HEAD; *switch to the branch to be merged into then execute* :
```shell
git merge [--no-ff|--squash] <branchName>
```
* By default if the merge succeeds it automatically does a commit. This can be overridden by the --no-commit flag.
* When merging from a public branch (that is a branch that someone else contributed to) one should use the --no-ff flag as this will guarantee a commit message into the repository. Allowing one to go back through the history later on and see where things came from. 
* When merging into a public branch from a private feature branch it is often useful to summarize the numerous commit messages one did while building the feature into a single message which introduced the feature to the repository. This is done with the --squash flag. 

Incorporate changes from another repository which were obtained by a git fetch (similar to what a git pull does):
```shell
git fetch <remote>
git merge
```


## git mv
* To move or rename a file:
```shell
git mv <file_from> <file_to>
```

## git pull
Similar to fetch the following command will automatically fetch and then merge a remote branch into the current branch:
```shell
git pull
```

## git push
Push one's changes to the remote repository
* The server providing the remote must supply write access
* No one else has pushed data to the remote repository in the meantime
  * The remote changes will need to be pulled down and merged into the local repository before they can be pushed back up to the origin.
  * Both of the following pushes the master branch to the origin server:
  ```shell
  git push origin master
  git push origin
  ```

Tags are not automatically pushed to remotes via git push. This must be done explicitly:
```shell
git push origin [tagName]
```

Push a branch up to a remote 
```shell
git push origin [branchName]

# Use this if the local branch name is not what is to be used
# on the public repository
git push origin [localBranchName]:[remoteBranchName]
```
* In this case origin is the name/reference to the server
* Local branches are not automatically synchronized and need to be done explicitly.

Delete a remote branch use the following obtuse syntax:
```shell
# Original obtuse syntax
git push [remoteName] :[remoteBranchName]

# Cleaner syntax introduced in v1.7.0
git push origin --delete <branchName>
```

Use the -f force flag to override the repository's behavior of requiring one to be up to date with the remote before the push. The flag will cause commits to be lost. This is useful when one mistakenly pushes a set of commits to a branch and then needs to reset the branch to a previous state then push the reset branch to the server.


## git rebase
Similar in merge in that it takes changes from one branch and puts them into another; this is done by replaying a series of commits. Extremely useful when on a long lived feature branch and one needs to pick up changes from the sources it was branched from. A swiss army knife for more powerful merging, as with all knives it is sharp... 

**GENERAL RULE OF THUMB… DO NOT REBASE COMMITS THAT HAVE ALREADY BEEN PUSHED TO A SHARED REPOSITORY**


Take changes from a main line and integrate them into a branch. This makes the history cleaner when the branch is merged back onto the mainline (e.g. master). 

```shell
git checkout <branchName>
git rebase master

# Or all on one line
git rebase master <branchName>
```

Take the changes associated with branch2 which is based on branch1 and  integrate them into the main line (e.g. master) use the onto flag from within branch2:
```shell
git checkout <branchName2>
git rebase --onto master <branchName1> <branchName2>
git checkout master
git merge <branchName2>
```

Squash all commits on the same branch into a single commit. 
```shell
git rebase -I HEAD~n
```
* n = number of commits to combine
* It will bring up an editor, pick the first then squash the rest. When the file is written and editor closed another editor session will appear which allows for ammending and/or removing the comments. 
* This is great if one makes many commits when developing a file then at the end wants to summarize all of those commits into a single change before pushing to the origin.
  * A better way is to use git merge --squash <branchX> to merge all of the changes from branchX onto the current branch while squashing the commit history into 1 log.

## git reset
Reset the current HEAD to to a specified state

To reset a branch to exactly match a remote branch. The following works well if one forgot to work off of a feature branch and applied changes to a remote branch by accident (forcing a pull which will cause a merge). 
```shell
git checkout myBranch
# To save the changes to another branch just in case one wants them
git commit -a
git branch myBranch_saved

# Reset the branch back to the origin
git reset --hard origin/myBranch
```

If the changes were mistakenly pushed to the remote repository, after the reset, use the --force flag at the end of the line to push the changes. e.g. git push origin myBranch --force
		

## git revert
Undoes a single commit. Instead of removing the commit the commit is undone and appends a new commit with the results. 
```shell
git revert <hash/tag>
```

This is no panacea, if there were other changes to the files which are affected by the revert then there will be conflicts which will need to be resolved…

Revert for single commits is simple and can be thought of an undo button; however for merges it is a totally different story. Reverting a merge can be done but is not straightforward… Merge wisely or the consequences will be an hour of ones life gone to figuring this out and the uneasyness of not knowing how things are going to turn out when the merge needs to happen at a later time… To re-merge one needs to revert the revert… argh More info [here](https://git-scm.com/blog/2010/03/02/undoing-merges.html)


## git rev-parse
Used to obtain the SHA1 of a given local tag or branch.
```shell
git rev-parse HEAD

# Or specify another branch
git rev-parse master
```

To obtain the SHA1 tag of a remote repository use ls-remote
```shell
git ls-remote <URL>
git ls-remote
```

## git remote
To determine the remote repository associated with the local repository
```shell
git remote -v
```

Add a new remote repository
```shell
git remote add <remoteShortName> <url>
```
* remoteShortName is simply an alias for the remote which can be used when referencing it in git commands.
* url is the path to the remote. Examples include:
  * git@gitserv.com:mojo/foo.git
  * /nfsserv/johndoe/project/project.git

To remove a remote repository
```shell
git remote rm <remoteShortName>
```


## git rm
* To remove a file and no longer track the file in source control use Git's rm command then commit the change:
```shell
git rm <file>
git commit
```

* To remove a file which has been added to the staging area but not committed
  one must force the removal with -f:
```shell
git rm -f <file>
```
_Notice that since no changes are being made to the repository then a commit is not necessary (there is nothing to commit)._

* To remove a file which was mistakenly added to the staging area (cache) but keep the working copy use the --cached option:
```shell
git rm --cached <file>
```


## git show
Show information about a git object; including the name, contact information, and tagging message. An object can be a blob, tree, tag, or commit:
```shell
git show <object>

# Show a file from another branch:
git show [branch]:[file]
```


## git stash
To switch branches without committing work the following command takes the modified tracked files and saves them on a stack of unfinished changes that can be reapplied later:
```shell
git stash
```

 To see what has been stashed:
 ```shell
 git stash list
 ```

To reapply the stashed changes :
```shell
git stash apply
```

It is also possible to do all sort of funky things like apply stashed changes to another branch and stash multiple changes and apply them out of order. See Pro Git, Git Tools : Stashing for more information

I have come to prefer an alternate method when I have to move from one branch to another in the middle of doing work. If I am not ready to commit my changes then I simply create a branch where I am and then merge it back in later (git checkout -b branchName )


## git status
Check on the status of the files 

To simply determine which files are in which state
```shell
git status
```
* States :
    * Untracked                 : Files not in source control
    * Not staged (Modified)     : Files tracked by git that have been modified but not added.
    * Staged (To be committed ) : Files ready to be committed into source control. 
    

## git tag
List all of the available tags. 
```shell
# Lists all tags in alphabetical order
git tag

# List tags for a particular pattern, supply the list pattern flag
git tag -l "ver1.2.*"
```

Create a tag 
```shell
# Create a tag at the head of the local repository
#Using tags in this way stores the tagger name, contact information, and the tagging message. 
git tag -a <tagName> -m "<taggingMessage>"

# Create a tag given a previous commit:
git tag -a <tagName> -m "<taggingMessage>" <commitChecksum>
```

Tags are not automatically pushed to remotes via git push. This must be done explicitly:
```shell
git push origin [tagName]
```


# Tasks

## Import Existing Sources Into a New Repository
1. Create the repository in one's favorite cloud repo (git-hub, bit-bucket, kiln, etc…), obtain the URL of the created repository (e.g. ssh://wpmackenzie@bitbucket.com/newrepo)
2. cd newrepo where newrepo contains the sources to be imported
3. Execute teh following git commands
```shell
	git init
	git add .
	git commit -a
	git remote add origin ssh://wparkermackenzie@bitbucket.com/newrepo
	git pull origin master
	git push origin master
```
	

## Create Branch from Tag
```shell
# Find the tag to create the branch from
git tag|sort

# Move gits focus onto the sources associated with the tag. 
git checkout <tagname>

# Create a new branch from the sources associated with the tag and move 
# the focus of git to this new branch.
git checkout -b <branchName>

# ...Make the changes...

# Commit the changes
git commit -a

git push origin <branchName>
git tag -a <newTagName>
git push origin <newTagName>
```


## Rename a tag
```shell
	git tag <newname> <oldname>
	git tag -d <oldname>
	git push origin --delete <oldname>
	git push origin <newname>
```
	
## Rename a branch
**Can't really be done remotely. Best that can be done is to rename it locally, remove the old branch remotely, then push the new branch.**
```shell
git checkout <oldname>
git branch -m <oldname> <newname>
git push origin --delete <oldname>
git push origin <newname>
```
	
**Argh… notice that renaming a branch has the oldname newname swapped from the process of renaming a tag…**
	
	
## Create a temporary change/feature branch
In the world of GIT almost every change is a branch which makes most branches temporary. In this case a change is made on its own branch, unit tested, then merged back into one or more release branches. In this case the branch used to unit test the change can be removed once the change has been merged onto one or more of the release branches.
```shell
# Move git's focus to the sources associated where the unit 
# change is to be made.
git checkout <tagName>

# Create a new branch from the sources associated with tagName 
# and move the focus of git to this new branch.
git checkout -b <unit-branchname>

# ...Make the changes...

# Commit the changes
git commit -a

# Test the changes…iterate through Make the changes until happy ...

# Checkout the tag name associated with the release branch 
# to place the change, or put it onto the master branch.
git checkout <releaseTagName>

#  Merge in the changes from the branch
git merge <unit-branchname>

# Check and test the merge...

# No longer need the temporary branch where the original changes were made
git branch -d <unit-branchname> 

# Push the changes to the origin "pristine" repository
git push origin 
```

## Merge individual changes into a previous branch
Here changes were made to newer version of the repository, those changes need to be back ported to an older branch. The merge and rebase commands do not work as they try to merge too much .
For example :
```
                           origin/master
                                ↓
	(C0)<-(C1)<-(C2)←(C3)←(C4)
	       ↑                ↑
	      (C1')            (C5)
	       ↑                ↑
   origin/bugfix_branch  remote/master
```
In this case someone made a change (C5) on the master branch. However, after making the change it was concluded that the change also needed to get back ported to the bugfix_branch. In this case the following commands take/cherry-pick the changes from (C5) and bring them onto the bugfix_branch:
	
```shell
git fetch remote/master

# Create a tracking branch
git checkout -b bugfix_branch origin/bugfix_branch

git cherry-pick -x remote/master

# Follow the directions; if there are conflicts, fix the conflicts then add them.
```

## Look at the differences between 2 branches
```shell
# This will show a textual diff of the 2 branches
git diff branch1..branch2
```
Or for a graphical representation (meld is my favorite...):
```shell
# This will use the diff tool configured for the user (git config --global diff.tool)
git difftool branch1..branch2
```

What changes do I have which have not been pushed to the origin
```shell
git diff origin/master..HEAD
```

## Debugging Access to a Repository
Working with bit bucket, git hub, and fogbugz; from time to time access to the remote repository will be less than stellar. When logging a support ticket the following command can be helpful
```shell
date; GIT_TRACE=2 GIT_CURL_VERBOSE=2 GIT_TRACE_PERFORMANCE=2 GIT_TRACE_PACK_ACCESS=2 GIT_TRACE_PACKET=2 GIT_TRACE_PACKFILE=2 GIT_TRACE_SETUP=2 GIT_TRACE_SHALLOW=2 git clone <REPOSITORY>;date

# Where <repository> is the repository to be cloned. 
```

# Work Flows 

## Integration-Manager Work Flow : As a manager
* Used when managing a project and working with other developers which contribute to the project. 
* The manager and contributors have clones of the blessed repository which stores the project; however, only the manager may push to the blessed repository.
  * Processes :
    * Integrating a contributors changes
       1. Contributor and manager come to agreement on the change.
          * Any code change whether a change to the design or a simple bug fix requires both parties to be in agreement at a design level as to the change. Also discussed are the timeframes involved, which includes the expectations of the maintainer to interact and review the code.
       2. Receive notification from the contributor that they are ready for a code review and to have the code integrated into the blessed repository.
       3. Add the contributor's repository as a remote if it was not previously added already and fetch the changes to that remote.
       ```shell
       git remote add <remoteShortName> <url>
       git fetch <remoteShortName>
       ```
       4. If integrating on a branch switch the local repository to the branch.
       ```shell
       git checkout <localBranchName>
       ```
       5. Gather the list of files which contain differences between the manager's copy of the sources and that of the contributor.
       ```shell
       git diff --name-status <remoteShortName>/<remoteBranchName>
       ```
         * The codes are as follows :
           * A.  File added
           * C.  File copied
           * D.  File deleted
           * M.  File modified
           * R.  File renamed
           * T.  Type changed
           * U.  Unmerged
           * X.  Unknown
           * B.  Broken
       6. Review the changes to each of the files. If changes need to be made communicate them with the contributor and go back to <1>
       ```shell
       git difftool <remoteShortName>/<remoteBranchName>
       ```
       7. Merge the changes into the local repository in such a way that the contributor's commit history (logs) will be overridden with a single well written log by the manager. If the auto merge fails, then git will indicate which files need to be merged by hand.
       ```shell
       git merge --squash <remoteShortName>/<remoteBranchName>
       ```
       8. Test the changes. If they do not work, communicate with the contributor and go back to <1>
       9. Update and commit any additional files which are associated with the change. This is usually documentation.
      10. Commit the merged changes and any additional documentation changes to the local repository. When the editor pops up, remove the squashed commit message(s) with a single well written summary.
      ```shell
      git commit -a
      ```
      11. Push the changes to blessed repository (assuming origin indicates the blessed repository...)
      ```shell
      git push origin <branchName>
      ```
      12. If necessary tag the branch and push the tag to the blessed repository:
      ```shell
      git tag -a <tagName>
      git push origin <tagName>
      ```
  * Integrating changes with other managers 
    * If there is more than 1 manager responsible for the repository
       1. Fetch all changes from the blessed repository
       ```shell
       git fetch origin
       ```
       2. Merge local changes with that of blessed repository
       ```shell
       git status
       git merge origin/<branchName>
       ```

## Integration-Manager Work Flow : As a contributor
* Processes :
  1. Contributor and manager come to agreement on the change.
    * Any code change whether a change to the design or a simple bug fix requires both parties to be in agreement at a design level as to the change. Discussion should include :
      * Timeframes involved
      * What are the expectations of the maintainer 
      * What are the expectations of the contributor
      * Dictates priority given to code review and integration
      * If the work is to be done on a branch and the branch name
  2. Fetch the latest changes from the blessed repository
  ```shell
  git fetch origin
  ```
  3. If necessary create a new branch
  ```shell
  git branch <branchName>
  ```
  4. If necessary switch the reposiory to the branch to be worked on
  ```shell
  git checkout <branchName>
  ```
  5. Modify the code; committing to the local repository as deemed necessary, document, and test the change.
    * Prior to committing the change it is recommended that the files are checked for annoying whitespace errors and fixed before hand.
    ```shell
    git diff --check
    ```
  6. When the changes are functional and ready for integration make sure everything was committed and send the integration manager the URL with a request to review and integrate the change.
    * To add any new files : 
    ```shell 
    git add <fileName>
    git commit -a
    ```
  7. DO NOT remove the branch or delete the sources until the changes have either been integrated with the blessed repository or there is agreement to abandon the work. That which is not on the blessed repository does not remain...


## Centralized Work Flow
Everyone has a clone of the blessed repository which stores the project and everyone may push to it. 

This is the best way of working fast in small teams where trust is had amongst team mates. 

Processes :
* Contribute and integrate changes to the centralized repository
  1. Clone the repository if not done so already. Unlike other source control systems, this is not done that often; almost all work on all branches is done in a single local repository.
  ```shell
  git clone <repository> <directory>
  ```
  2. Fetch the latest changes from the blessed repository
  ``` shell
  git fetch origin
  ```
  3. Working on a branch
    * Determine if a new branch or local tracking branch need to be created.
      * If the work is new and it might be long lived and/or there are multiple contributors then create a new branch
      ```shell
      git branch <branchName>
      ```
      * If there is an existing branch that needs to be worked on and is this the first time work has been done on that branch in the local repository, if so a local branch needs to be created which will track the remote branch. In this workflow, remoteName will almost always be origin.
      ```shell
      git checkout --track [remoteName]/[remoteBranchName]
      ```
      * If working on a branch, switch the local repository to the branch 
      ```shell
      git checkout <branchName>
      ```
      * Modify the code; committing to the local repository as deemed necessary.
      * If working off of a long lived topic branch (bug fix/new feature) it may be necessary to integrate the changes from the main line (master) branch into the topic branch. 
        * Go to the master branch (assuming there are no local changes) and fetch the latest changes from the server:

        ```shell
        git checkout master
        git fetch origin
        git merge origin/master

        # Go back to the branch
        git checkout <branchName>
        
        # Tag the sources in the branch to put a place holder where the
        # master branch was integrated.
        git tag -a <tagName>

        # This will rewind all of the changes currently made to the branch
        # then apply the changes in the main line. Lastly, the changes
        # associated with the branch will be merged in.
        git rebase master
        ```
         
      * If working on a branch and want to push the branch changes to the branch in the centralized repository :
      ```shell
      # Remember tags are not automatically pushed they must be done manually.
      git push origin [branchName]
      ```
      * When it is time to merge the changes back into the main line (master) branch.
        * Tag the sources at the point they were integrated back into the main line. 
        ```shell
        git tag -a <tagName>
        ```

  4.On the master branch obtain the latest changes from the server

  ```shell
  git checkout master
  git pull origin

  # Or via a fetch and rebase
  get fetch origin
  git rebase origin/master
  ```
  5.Merge the changes in the branch on to the mainline (master) branch. Most of the time one will want to collapse all commit history on the branch into a single commit onto the main line. However, for those exceptions to the rule one can leave out the --squash option. 

  ```shell
  # Merges the changes from the branch onto the main line branch in
  # a single commit
  git merge --squash <branchName>
  git commit -a
  ```
  6.When the code has been tested, documented, all (if any) reviews are complete, and everything has been committed locally then the changes can be pushed to the centralized repository. When working in a team setting on a centralized repository, it is possible that the commit will fail as another team member may have pushed a change.
  ```shell
  git push origin master
  ```
  7.To tidy things up a bit, consider deleting unused branches and tags once the sources have been integrated.

## Git Flow
Typically a centralized repository with very specific branch naming conventions with a well defined process. More notes later on this great way to take source control out of the wild west by adding a touch of process. 







