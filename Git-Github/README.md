
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



# Clone
git clone https://github.com/zivkashtan/course.git

- Remove Origin
ubuntu@ip-172-31-46-163:~/mygitrepo$ git remote remove origin
- Add Origin
$ git remote add origin https://github.com/lonynamer/time-tracker.git
- Push Origin
$ git push --all origin
Username for 'https://github.com': lonynamer
Password for 'https://lonynamer@github.com':
Counting objects: 51, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (32/32), done.
Writing objects: 100% (51/51), 11.02 KiB | 0 bytes/s, done.
Total 51 (delta 7), reused 0 (delta 0)
remote: Resolving deltas: 100% (7/7), done.
To https://github.com/lonynamer/time-tracker.git
 * [new branch]      master -> master



- Remove from Staging
$ git reset HEAD test.txt
$ git add .
-  -a commit and add but adds change, new files you have to do add. -m message 
$ git commit -a -m "The changes I did."
$ git push --all origin
- show origin
$ git remote show origin



*** BRANCHING
*** Branch > Snapshots. ordered sets of commits. Whenever you do commit, Master points to last brach.
*** You can create branch without changes. 
A brach will point to my new branch and master will point to my last change. 
When you turn back from branch A, you turn back to situation in M.

- Creates new branch if exists, else it creates new.
$ git checkout -b new_web_handler
- List branches locally 
$ git branch -a
- Push Branch to upstream
$ git push origin test_feature
$ git push --set-upstream origin

**** Merging
- You create branch of master, create target branch merge and solve conflicts and merge to master





##### REVERT BACK TO A COMMIT AND DELETE ALL HISTORY AFTER
---
- This will delete the commit history after the point you rolled-back.
- So be carefull.
- When you use this, create a new branch, rollback in the new branch and merge to the original branch.
---
```
git reset --hard f414f31
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
