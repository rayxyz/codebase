# git remote -v 
View existing remotes
```
> git remote -v
origin	https://github.com/abc/xxxxxx.git (fetch)
origin	https://github.com/abc/xxxxxx.git (push)
```
# git rename 
Rename an existing remote
```
> git remote rename origin old_origin
> git remote -v
old_origin	https://github.com/abc/xxxxxx.git (fetch)
old_origin	https://github.com/abc/xxxxxx.git (push)
```
# git add remote
Add a new remote, you can add credentials in the url
```
> git remote add origin https://username:password@github.com/abc/xxxxxx.git
> git remote -v
git remote -v
old_origin	https://github.com/abc/xxxxxx.git (fetch)
old_origin	https://github.com/abc/xxxxxx.git (push)
origin	https://username:password@github.com/abc/xxxxxx.git (fetch)
origin	https://username:password@github.com/abc/xxxxxx.git (push)
```
# git remote remove
Remove a remote
```
> git remote remove old_origin
> git remote -v
origin	https://username:password@github.com/abc/xxxxxx.git (fetch)
origin	https://username:password@github.com/abc/xxxxxx.git (push)
```
# git set-url 
Set new remote url
```
ray@ray-zdz:~/workspace/blogbak$ git remote -v
origin	https://gitclone.com/github.com/rayxyz/blogbak (fetch)
origin	https://gitclone.com/github.com/rayxyz/blogbak (push)

ray@ray-zdz:~/workspace/blogbak$ git remote set-url origin https://github.com/rayxyz/blogbak
ray@ray-zdz:~/workspace/blogbak$ git remote -v
origin	https://github.com/rayxyz/blogbak (fetch)
origin	https://github.com/rayxyz/blogbak (push)
```

# set tracking information `master` branch for the newly add remote
```
git branch --set-upstream-to=origin/master master
```
# git pull (without username and password promt)
```
> git pull
Already up-to-date.
```

# push a new branch to remote
```
git push -u origin ray
```

# view all branches
```
git branch -a
```

# delete a branch
```
git branch -d dev
```

# solve `error: Your local changes to the following files would be overwritten by merge`
```
git stash
git stash drop
git pull
```
Check out stash workflow: [https://css-irl.info/how-git-stash-can-help-you-juggle-multiple-branches/](https://css-irl.info/how-git-stash-can-help-you-juggle-multiple-branches/)

The local changes will be lost after above operations.

# restore deleted/misoperations of files
```
git reset HEAD filecoin-ffi/
```

# Push local new branch to remote and track
```
git push -u origin <branch>
```

# List configs
```
git config -l
```

# Create a branch and check it out.
```
git checkout -b ray
```

# Ignore some file modified and added it to the .gitignore but still showing under the command `git status`
```
git rm -r --cached github.com/golang/protobuf/protoc-gen-go/protoc-gen-go
```

# Reset add
```
git reset HEAD github.com/golang/protobuf/protoc-gen-go/protoc-gen-go
```

# Reset commit
```
git reset --soft HEAD^
```

# Credential caching
```
git config credential.helper store
```
## With specific time to expire (eg.: 2hrs)
```
git config --global credential.helper 'cache --timeout 7200'
```

# Merge
Merge branches fixes and enhancements on top of the current branch, making an octopus merge:
```
$ git merge fixes enhancements
```

Merge branch obsolete into the current branch, using ours merge strategy:
```
$ git merge -s ours obsolete
```

Merge branch maint into the current branch, but do not make a new commit automatically:
```
$ git merge --no-commit maint
```

# Force checkout branch
```
git checkout -f another-branch
```

# Change the remote
```
git remote
git remote set-url origin git@192.168.1.252:wangrui/sdmicro.git
```

# Forcefully overrite the remote repo with local one
```
git push origin dev -f
```

# Check size of Git proj
```
git count-objects -vH
```

# assume file unchanged(actually it has been changed)
git update-index --assume-unchanged vendor/github.com/golang/protobuf/protoc-gen-go/protoc-gen-go
# and track the changed file again
git update-index --no-assume-unchanged vendor/github.com/golang/protobuf/protoc-gen-go/protoc-gen-go

# Change user.name
```
 git config --global user.name "rayxyz"
```

# Stashing
```
Stashing before switch branches to avoid errors when switch to another branch, but the current changes are not committed.
```
## Stash
```
git stash
```
## Stash list
```
git stash list
```
```
stash@{0}: WIP on ray: d805c16 add user appmodule rel feature
stash@{1}: WIP on ray: d805c16 add user appmodule rel feature
```
## Apply a stash
```
git apply
git apply stash@{1}
git stash pop
```
## Drop a stash
```
git stash drop stash@{0}
```
# Unmerge
```
git merge --abort
```
# Checkout a remote branch
```
git checkout -b newlocalbranchname origin/branch-name

Or

git checkout -t origin/branch-name

The latter will create a branch that is also set to track the remote branch.me
```
