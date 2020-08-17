# SCP
```
scp ahz_common root@192.168.1.164:/ahezime/bin
scp username@remote:/file/to/send /where/to/put
scp username@remote_1:/file/to/send username@remote_2:/where/to/put

scp ahezime@46.12.97.215:/root/dbbackups/ahezime_workflow.sql ./ahezime_workflow.sql

<> copy a remote folder to local directory
root@ahezime:~# scp -r root@1.2.3.4:/root/file ./file

<> copy local folder to remote directory
> scp -r ./dist ahezime@1.2.3.4:/root/abc/web/platform/
```
