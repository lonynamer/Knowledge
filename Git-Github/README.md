
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

##### SHOW COMMITS
`git log`
---
Will show you all commits in the current branch(master)
---
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





