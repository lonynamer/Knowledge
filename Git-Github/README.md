
##### What is GIT ?
---
- Git is a multi-branched distributed source control management system developed by Linus Tovarlds.
- The main difference from SVN is; it's snapshot file system structure instead of using directories and duplicates under branches.
- By this property and some compression methodologies reduces the data size drastically.
- As data size is reduced; In Git world, developers don't work in front of a server, each developer `clone` the whole remote repository as local repository.
- Developers work on local repo and can `pull` others changes and `push` their changes.
- This way the source code is distributed and also collaborative. 
- `Master` is the main branch. Source code can be branched like `Development`, `Staging`, `Feature`, `BugFix`, `Release` for different scenarios of code change.
- This is the recommended way to work.
- In the end, changes from the branches can be merged to another branches and `tag` as a version can be done.
- All this changes collaboratively can be pushed to a remote origin server.
- Once a repository is cloned only chages will be pushed or pulled and it will work light-weight.
---

##### What is GITHUB ?
---
- `GITHUB` is a remote git repository for open-source projects.
- `BitBucket` is a competitor with same principles.
- `GitLab`, `GitHub Enterprice` are some other solutions.
- For creating private repositories.
- For Organizations, you need to use `GitHub Enterprice`, `BitBucket Enterprise`, maybe `GitLab` for having more control on security, privacy, authorization, repository and branch permissions.
- Another option may be using LDAP with ssh or https.
- GIT can contact remote repo over ssh or https securely.
---

##### GIT REPOSITORY SECTIONS AND STATES
---
- States: `Untracked` `Tracked` `Modified` `Unmodified` `Staged`
- Sections: `Workspace` `Staging Area` `Commit Area`
- A local repository is just a directory initialized specially by `.git` directory.
- By default the files are `untracked` under your `workspace` where you see the files.
- You have to add the file/s or dir/s to `staging area`, so the changes will be tracked and files will be staged.
- When you do a change to a `Staged` file, it's state will change to `Modified`
- A modified file should be `Staged` by `add` again or `-a` flag of `commit` handles this.
- `-a` flag does not handle newly added files.
- When you delete a file also need to use `-a` flag of commit.
- When you `commit` only the files in `tracked` and `staged` status will be commited to `Commit Area`
- When do `push` to remote origin repo, those committed changes will be pushed to remote repo.
- It's adviced to do a `pull` to have, all the changes of others first to solve conflicts and then push to central repository.
- Listing, deleting, removing, rollback from staging area or commits all possible from on all branches.
---

##### INSTALL GIT
On Ubuntu/Debian
```
apt-get update
apt-get install git-core
apt-get install git  # Also fine.
```
On Centos
```
yum install git
```
On Windows
```
- Download from https://git-scm.com/download/win
- Install full and use `Git Bash` which is recommended.
- You can also use `Git CMD` or `Git GUI`
```

##### INSTALLING GITHUB ENTERPRISE ON AWS
---
- We will use regular free Github. Can install just to learn later.
- https://enterprise.github.com/resources
---
```
aws cloudformation create-stack \
  --region eu-west-1 \
  --stack-name "GitHubTrial" \
  --template-url https://github-enterprise.s3.amazonaws.com/cloudformation/trial-1533312825.template \
  --parameters ParameterKey=Instance,ParameterValue=r3.large \
               ParameterKey=Data,ParameterValue=50
```

##### OPEN A FREE GITHUB ACCOUNT
---
- https://github.com
- Follow the steps and create an account by default addresses as https://github.com/your-username
- You can also use bitbucket. 
---

##### CONFIGURE YOUR LOCAL IDENTITY
```
git config --global user.name "Lony Namer"
git config --global user.email "namer.lony@gmail.com"
```
- This configuration will create a .gitconfig file under your home directory like below.
$ cat /home/ubuntu/.gitconfig
```
[user]
        name = Lony Namer
        email = namer.lony@gmail.com
```

##### CREATE AND EMPTY LOCAL REPOSITORY `mygitrepo`
```
cd ~ 
mkdir mygitrepo
cd mygitrepo
git init
```
`Initialized empty Git repository in /mygitrepo/.git/`

##### CREATE AND ADD FILES TO STAGING AREA
```
touch file1 file2 file3
mkdir test
touch test/test1 
git add file1   # Add file
git add .       # Add All Files
```

##### SHOW STAGED FILES
`git status`
```
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   file1
        new file:   file2
        new file:   file3
        new file:   test/test1
```

##### REMOVE FILES FROM STAGING
```
git reset HEAD -- file1   # File or directory
git reset HEAD -- .       # All
```
Do a `git status`, you will see that files are not tracked or staged.
```
git status
```

##### COMMIT AND SAVE CHANGES
---
- `commit` means saving a snapshot of tracked and staged files.
- Before doing `commit`, you need to add the file to stagin as above.
- `Tracked` and `Staged` donot mean the same. 
- If a file is `Tracked` but `Modified` later, you need to stage it again.
- `-m` flag is message, plain text description of the changes. Always describe the changes.
---
Have to use `add` for new files.
```
git add .    
git commit -m "Added all the files."
```
If files are only modified.
```
echo "test modification" >> file1
```
This will handle the changes.
```
git commit -a -m "file1 changed, new line added."
```
Output:
```
[master 3ec5d23] file1 changed, new line added.
 1 file changed, 1 insertion(+)
```

##### SHOW COMMITS ON THE CURRENT BRANCH
```
git log
```
Output:
```
commit 3ec5d239ffde3cdb02d2a2740b738d8592afee13 (HEAD -> master)
Author: Lony Namer <namer.lony@gmail.com>
Date:   Mon Oct 1 22:35:19 2018 +0300

    file1 changed, new line added.

commit 43ddfc4c8eed65cf950e1067c16e42b53e03b39c
Author: Lony Namer <namer.lony@gmail.com>
Date:   Mon Oct 1 22:32:12 2018 +0300

    All files added.
```

##### CREATE AN EMPTY REMOTE REPOSITORY  AT GITHUB (Don't Initialize) AND PUSH THE CHANGES
---
- https://github.com/username/reponame
- Mine: https://github.com/lonynamer/mygitrepo
- `origin` is just a label, you can set different.
---
- Add remote repository 
```
git add remote origin https://github.com/lonynamer/mygitrepo.git
# OR
git add remote origin git@github.com:lonynamer/mygitrepo.git
```
- Push the changes. As it's the first time and remote repo has no branches even master. Credentials required.
```
 git push --set-upstream origin master
```
- Next time, you can push the changes like this.
```
git push
```
- Pushing all the branches first time to empty repo.
```
git push --all --set-upstream origin
```
- Pushing all the branches
```
git push --all
```

##### SHOW REMOTE REPOS
```
git remote show
```
Output:
```
origin
```
```
git remote show origin
```
Output:
```
git remote show origin
* remote origin
  Fetch URL: https://github.com/lonynamer/mygitrepo.git
  Push  URL: https://github.com/lonynamer/mygitrepo.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

##### BRANCHING AND CHECKOUT
- Creating a branch from other branch means getting ordered set of commits getting under another branch.
- Create a brach
```
git branch development
```
- List the branches
```
git branch -a
```
- Checkout to branch
```
git checkout development
```
- Create a branch and checkout.
```
git checkout -b development
```
- Push a branch
```
git push origin development
```

##### MERGING
- Creating branch of master, create target branch merge, solving conflicts and merging back to master is a good practice.
```
git checkout master
git merge development
```
It will ask the merge reason vi style. After `:wq!` `development` branch will be merged to `master` .

##### CLONE A REMOTE REPO
- Just for testing remove the local repository. Clone back from remote repo.
```
cd ~
rm -rf mygitrepo
git clone https://github.com/lonynamer/mygitrepo
# After Cloning, Check
cd mygitrepo
git remote show origin
```

##### REVERT BACK TO A COMMIT AND DELETE ALL HISTORY AFTER
---
- This will delete the commit history after the point you rolled-back.
- So be carefull.
- When you use this, create a new branch, rollback in the new branch and merge to the original branch.
---
```
git reset --hard f414f31
```
##### REVERT BACK BY CREATING A NEW COMMIT END UNDOING CHANGES
- Create a new commit that undoes the changes in a commit.
```
git revert 08bdf23bcebdab929884d6f1f2ed3e0f797831d2
```

##### REVERT BACK BY CREATING A NEW COMMIT FROM AN OLD COMMIT 
- This one also adviced by using a bracnh.
```
git reset --hard f414f31
git reset --soft HEAD@{1}
git commit -m "Roll Back To f414f31 and new commit."
```

##### CLEAN UNTRACKED FILES
---
- First see what will be removed
- It may clean also ignored files as 
---
```
git clean -n
```
- Delete
```
git clean -f
```


### GIT CHEAT SHEAT

##### Git Basics
- Create empty git repo
```
git init directoryname
```
- Clone a remote repo
```
git clone https://github.com/lonynamer/mygitrepo
```
- Stage a file or directory
```
git add directoryname
# All file in currect directory
git add .
```
- Commit
```
git commit -m "Change description message"
# By adding changed files.
git commit -a -m "Change description message"
```
- List staged, unstaged and untracked files.
```
git status
```
- Display commit history
```
git log
```
- Show unstaged changes between working directory and index
```
git diff
```

##### Undoing Changes
- Create a new commit that undoes the changes in a commit.
```
git revert 08bdf23bcebdab929884d6f1f2ed3e0f797831d2
```
- Remove a file from staging but leave in working dir.
```
git reset myfile
# All Files
git reset .
```

##### Rewriting Git History
- Replace last commit with staged changes and last commit combined.
```
git commit --ammend
```
- Rebase current branch. It can be commit id, branch, tag, or relative reference to HEAD.
```
git rebase master
# OR 
git rebase 08bdf23bcebdab929884d6f1f2ed3e0f797831d2
```
- Show log of changes
```
git reflog --all --relative-date
```
##### Branches
- List Branches
```
git branch
```
- Create And Checkout Branches
```
git checkout -b branchname
```
- Merge a bracnch to the current branch
```
git merge branchname
```

##### Remote Repositories
- Add a remote repo.
```
git remote add origin https://github.com/lonynamer/mygitrepo.git
```
- Fetch a specific branch
```
git fetch origin branchname
```
- Pull remote repos current branch
```
git pull origin
```
- Push a branch
```
git push origin
# Also fine if origin is set.
git push 
```
- Push all branches
```
git push origin --all
# Also fine if origin is set.
git push --all
```

##### git config
- Set author name current user
```
git config --global user.name "Lony Namer"
```
- Set author email for current user
```
git config --global user.email namer.lony@gmail.com
```
- Set text editor used for merges and conflicts etc for all users.
```
sudo git config --system core.editor vi
```
- Edit the system configuration file
```
sudo git config --system --edit
```
- Edit the global configuration file for user
```
git config --global --edit
# OR
vi ~/.gitconfig
```


