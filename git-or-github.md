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

# delete a branch
```
git branch -d dev
```
