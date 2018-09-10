JOHNB - GIT
18.197.145.170
20	לוני נמר	ubuntu	https://s3.eu-central-1.amazonaws.com/jb-artifacts/jb_ubuntu.pem	18.197.145.170


- Open Github Account
https://github.com/GabrielWM/JB-SSH/blob/master/jb-cours-ssh.xlsx


- INSTALL
apt-get update
apt-get install git-core

- CONFIGURE YOUR IDENTITY
root@ip-172-31-46-163:~# git config --global user.name "Lony Namer"
root@ip-172-31-46-163:~# git config --global user.email "namer.lony@gmail.com"

- Create Repository 
root@ip-172-31-46-163:~# cd /mygitrepo/
root@ip-172-31-46-163:/mygitrepo# git init
Initialized empty Git repository in /mygitrepo/.git/

root@ip-172-31-46-163:/mygitrepo# cat /root/.gitconfig
[user]
        name = Lony Namer
        email = namer.lony@gmail.com



- Clone locally
root@ip-172-31-46-163:/mygitrepo# git clone git@bitbucket.org:automatitdevops/country/country.git

# git status # (will show what changed.)

Git Area divided to 3 sections  in localrepo
- Working Directory: Files. change files but did a change on a file but don't want it on staging
# git add file (adds to staging)
# git checkout file (removes from staging)
- Stage: From working dir moves to staging and commit works on staging.
# git commit -m "explenation" # (adds to repository)
- Repository

# git push # (push to remote)
# git pull # (pull from remote)



> git add . 
> git reset HEAD . 

-----------------------------

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


- Create 2 file
$ echo "I am a Git Overlord there!" > test.txt
$ echo "I am a Git Overlord there!" > test1.txt
- Status shows 2 untracked files
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
- Add to Staging
$ git add test.txt
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


aws cloudformation create-stack \
  --region eu-west-1 \
  --stack-name "GitHubTrial" \
  --template-url https://github-enterprise.s3.amazonaws.com/cloudformation/trial-1533312825.template \
  --parameters ParameterKey=Instance,ParameterValue=r3.large \
               ParameterKey=Data,ParameterValue=50


