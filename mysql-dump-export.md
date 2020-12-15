# backup
```
mysqldump -u root -p --opt your_database > your_database_bak.sql
```

# dump/export limited rows of a table
```
mysqldump -u root -p --opt --where="1=1 and method=5 order by id desc limit 10 " my_awesome_database hello_table > dump.sql
```
