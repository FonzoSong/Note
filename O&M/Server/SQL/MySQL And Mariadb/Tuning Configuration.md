```cnf
[mysqld]
#支持大小写
lower_case_table_names = 0
#数据库 innodb 缓存(3/4物理内存大小)
innodb_buffer_pool_size = 4G
#数据库 log 缓存
innodb_log_buffer_size = 64M
```