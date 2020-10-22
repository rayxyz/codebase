# du => View size of directory
```
du -sh ./
du -d 1 -h ./
```

# rsync
```
rsync -zahP root@10.42.12.92:/data/sdb/lotus-user-1/.lotus ./
```

# Symbolic link (soft link)
```
ray@rayvm01:/usr/local/jdk1.8.0_271/bin$ sudo ln -s /usr/local/jdk1.8.0_271/bin/java /usr/local/bin
```
