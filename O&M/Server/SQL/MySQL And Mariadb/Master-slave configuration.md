##### master
###### bash
```bash
[root@xnode2 ~]# echo -e  '[mysqld]\nserver_id=1\nlog_bin=mysql-bin' >> /etc/my.cnf
[root@xnode2 ~]# mysqladmin password 000000
[root@xnode2 ~]# systemctl restart mariadb
```
###### sql
```sql
MariaDB [(none)]> grant replication slave on *.* to 'user'@'192.168.200.13' identified by '000000';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.000 sec)
```
##### slave 
###### bash
```bash
[root@xnode3 ~]# echo -e  '[mysqld]\nserver_id=2\nlog_bin=mysql-bin' >> /etc/my.cnf
[root@xnode3 ~]# mysqladmin password 000000
[root@xnode3 ~]# systemctl restart mariadb
```
###### sql
```sql
MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST='192.168.200.12',MASTER_USER='user',MASTER_PASSWORD='000000';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: 192.168.200.12
                   Master_User: user
                   Master_Port: 3306
                 Connect_Retry: 60
               Master_Log_File: mysql-bin.000001
           Read_Master_Log_Pos: 771
                Relay_Log_File: xnode3-relay-bin.000002
                 Relay_Log_Pos: 1070
         Relay_Master_Log_File: mysql-bin.000001
              Slave_IO_Running: Yes
             Slave_SQL_Running: Yes
               Replicate_Do_DB:
           Replicate_Ignore_DB:
            Replicate_Do_Table:
        Replicate_Ignore_Table:
       Replicate_Wild_Do_Table:
   Replicate_Wild_Ignore_Table:
                    Last_Errno: 0
                    Last_Error:
                  Skip_Counter: 0
           Exec_Master_Log_Pos: 771
               Relay_Log_Space: 1380
               Until_Condition: None
                Until_Log_File:
                 Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File:
            Master_SSL_CA_Path:
               Master_SSL_Cert:
             Master_SSL_Cipher:
                Master_SSL_Key:
         Seconds_Behind_Master: 0
 Master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error:
                Last_SQL_Errno: 0
                Last_SQL_Error:
   Replicate_Ignore_Server_Ids:
              Master_Server_Id: 1
                Master_SSL_Crl:
            Master_SSL_Crlpath:
                    Using_Gtid: No
                   Gtid_IO_Pos:
       Replicate_Do_Domain_Ids:
   Replicate_Ignore_Domain_Ids:
                 Parallel_Mode: conservative
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
              Slave_DDL_Groups: 3
Slave_Non_Transactional_Groups: 0
    Slave_Transactional_Groups: 0
1 row in set (0.000 sec)

ERROR: No query specified
```

