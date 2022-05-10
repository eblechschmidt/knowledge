# Monitoring replication status of a msysql db with telegraf

Monitoring the replication status of a mysql database with telegraf is as easy as adding the following lines to the `telegraf.conf`:

```
[[inputs.sql]]
driver = "mysql"
dsn = "username:password@tcp(mysqlserver:3307)/"
[[inputs.sql.query]]
query="SHOW SLAVE STATUS"
measurement = "slave_status"
tag_columns_include = ["Master_Host"]
```

It only worked explicitely stating `tcp` as telegraf was not to determine protocol otherwise.

