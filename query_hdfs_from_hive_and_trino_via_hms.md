Create a file on a Dataproc node:
```bash
cat <<EOF > users.csv
id,name
1,John
2,Jane
3,Bob
4,Alice
5,Charlie
EOF
```

Load the file to hdfs:
```bash
hadoop fs -mkdir -p /user/yourusername/hdfs_directory/
hadoop fs -copyFromLocal /tmp/data.csv /user/yourusername/hdfs_directory/
```

Connect to hive:
```bash
hive
```

Create an external table:
```bash
-- Create an external table
CREATE EXTERNAL TABLE users (
  id INT,
  name STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/yourusername/hdfs_directory/';
```

Query it (with hive):
```sql
-- show see the 'users' table
show tables; 
-- query
select * from users;
```

Now connect to trino:
```
trino
```

Query it the same way:
```bash
Query it (with hive):
```sql
-- show see the 'users' table
show tables; 
-- query
select * from users;
```

As can be seen, Trino is leveraging HMS.
