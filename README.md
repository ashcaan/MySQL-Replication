# MySQL-Replication
We will provide step-by-step guide to achieve MySQL Replication with Master-Slave scneario.

Our Enviroment: 
Master = SRV1
Slave = SRV2

Step-1: Setting up MySQL conif files.
On SRV1 edit MySQL cnf file. usually located in /etc/my.cnf or /etc/mysql/my.cnf and add below code:

```shell
bind-address    = 0.0.0.0
log-error=/var/lib/mysql/srv1.err
performance-schema=0
#Replication Config#
server_id           = 1
log_bin             = /var/log/mysql/mysql-bin.log
log_bin_index       = /var/log/mysql/mysql-bin.log.index
relay_log           = /var/log/mysql/mysql-relay-bin
relay_log_index     = /var/log/mysql/mysql-relay-bin.index
expire_logs_days    = 3
max_binlog_size     = 100M
log_slave_updates   = 1
auto-increment-increment = 2
auto-increment-offset = 1
```
On SRV2 do the same thing for MySQL cnf file and add below code:

```shell
bind-address    = 0.0.0.0
log-error=/var/lib/mysql/srv1.err
performance-schema=0
#Replication Config#
server_id           = 2
log_bin             = /var/log/mysql/mysql-bin.log
log_bin_index       = /var/log/mysql/mysql-bin.log.index
relay_log           = /var/log/mysql/mysql-relay-bin
relay_log_index     = /var/log/mysql/mysql-relay-bin.index
expire_logs_days    = 3
max_binlog_size     = 100M
log_slave_updates   = 1
auto-increment-increment = 2
auto-increment-offset = 2
```

Step-2: Creating replication user in MySQL
On SRV1 loging to mysql and run below queries:

```sql
CREATE USER ‘replication_user’@’SRV2-IP‘ IDENTIFIED BY ‘replication_password‘;
GRANT REPLICATION SLAVE ON . TO ‘replication_user’@’SRV2-IP‘;
FLUSH PRIVILEGES;
```
On SRV2 loging to mysql and run below queries:

```sql
CREATE USER ‘replication_user’@’SRV1-IP‘ IDENTIFIED BY ‘replication_password‘;
GRANT REPLICATION SLAVE ON . TO ‘replication_user’@’SRV2-IP‘;
FLUSH PRIVILEGES;
```
Restart MySQL service after making above changes.

Step-3: Transfer MySQL databases from Master to Slave

In order to create dump of all databases use below command:

```shell
mysqldump -u root -p --all-databases --master-data > /root/data.sql
```

we use --master-data to have record of master mysql-bin log file name and it's position. After that open data.sql file and make note of below line:

```sql
MASTER_LOG_FILE='mysql-bin.XXXX', MASTER_LOG_POS=YYYY;
```

Transfer the files with scp to the slave server. 

```shell
scp /root/data.sql root@SRV2:/root/dbissue/
```
After Transfering the dump restore it:

```shell
mysql -uroot -p < /root/data.sql
```

Step-4: Connecting slave to master

Loging to mysql and run below command to connect slave to master

```sql
CHANGE MASTER TO MASTER_HOST='SRV1', MASTER_USER='replication_user', MASTER_PASSWORD='replication_password', MASTER_LOG_FILE='mysql-bin.XXXX', MASTER_LOG_POS=YYYY;
```

Use correct XXXX and YYY that you got from data.sql file in above command.

Finally run below command to start replication:

```shell
START SLAVE;
```

Step-5: Check replication status

You can login to MySQL and use below command to view slave status:

```sql
SHOW SLAVE STATUS\G;
```