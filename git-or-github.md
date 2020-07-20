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
# set tracking information `master` branch for the newly add remote
```
git branch --set-upstream-to=origin/master master
```
# git pull (without username and password promt)
```
> git pull
Already up-to-date.
```
