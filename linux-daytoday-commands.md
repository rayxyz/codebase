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

# systemctl 
Enable service to auto start
```
systemctl enable v2ray
```

Disable service to not to auto start
```
systemctl disable v2ray 
```

View enabled system services
```
systemctl list-unit-files --state=enabled
```
