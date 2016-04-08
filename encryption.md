# MariaDB Server 10.1 Encryption

MariaDB Server 10.1 supports encryption for tables and InnoDB tablespaces. Encrypting your tables make it almost impossible to steal a disk and access the data as long as they don't steal the keys with it! Encryption works for XtraDB (the default), InnoDB, and Aria (`ROW_FORMAT=PAGE`). Please note the [current limitations](https://mariadb.com/kb/en/mariadb/data-at-rest-encryption/#limitations).

## Documentation
* [Data at Rest Encryption](https://mariadb.com/kb/en/mariadb/data-at-rest-encryption)

## Notes
* For ease of use (i.e. when you are deploying it), look at using the `/etc/my.cnf.d/enable_encryption.preset` - this will ensure you are running in an encrypted mode by default and you do not have to worry about not having all the options.

## Exercise: using file key management
1. `create database unenc;`
2. `use unenc;`
3. `create table t (id int, value varchar(255));`
4. `insert into t values (1,12345);`
5. `insert into t values (2,'abc12345');`
6. 
	MariaDB [unenc]> select * from t;
	+------+----------+
	| id   | value    |
	+------+----------+
	|    1 | 12345    |
	|    2 | abc12345 |
	+------+----------+
	2 rows in set (0.00 sec)

7. Exit the MySQL shell, visit `/var/lib/mysql`, run `strings` on t.ibd. You should be able to see the values above. In fact, if you run `strings` on the binary log, you should also see some values of interest (especially if you've annotated the binlog)
8. Let's encrypt! First we'll create keys: `openssl enc -aes-256-cbc -P -md sha1`

	[root@centos-2gb-lon1-01 ~]# openssl enc -aes-256-cbc -P -md sha1
	enter aes-256-cbc encryption password:
	Verifying - enter aes-256-cbc encryption password:
	salt=FF80D2DFC5451D11
	key=E17903125BDA67576ACDB1A6D769B5A64FE75981901087270A7027229C9CC700
	iv =3CFD2ACB8AC2F76E430EF65A35BACC23
	[root@centos-2gb-lon1-01 ~]# openssl enc -aes-256-cbc -P -md sha1
	enter aes-256-cbc encryption password:
	Verifying - enter aes-256-cbc encryption password:
	salt=89988BB3E09F5E08
	key=A5A0E914C1B0FF86C220783EF8E188B0454C804E685855520706FAB635947CC8
	iv =21F4ADE22380DABC18399874C64A174E

9. So lets make the keys.txt based on the above info:
1;21F4ADE22380DABC18399874C64A174E;A5A0E914C1B0FF86C220783EF8E188B0454C804E685855520706FAB635947CC8
10. You can additionally encrypt the keys, but for simplicity sake, this was not done here (FYI: `openssl enc -aes-256-cbc -md sha1 -k secret -in keys.txt -out keys.enc`).
11. Make sure the keys.txt is in `/var/lib/mysql` and then edit your `my.cnf` and in the `[mysqld]` group add:
	plugin-load-add=file_key_management.so
	file-key-management-filename = /var/lib/mysql/keys.txt
12. `show plugins` should now have the following output:
	| file_key_management           | ACTIVE   | ENCRYPTION         | file_key_management.so | GPL     |
13. Now let's verify:
	MariaDB [unenc]> show create table t;
	+-------+----------------------------------------------------------------------------------------------------------------------------+
	| Table | Create Table                                                                                                               |
	+-------+----------------------------------------------------------------------------------------------------------------------------+
	| t     | CREATE TABLE `t` (
	  `id` int(11) DEFAULT NULL,
	  `value` varchar(255) DEFAULT NULL
	) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
	+-------+----------------------------------------------------------------------------------------------------------------------------+

	MariaDB [unenc]> alter table t encrypted=yes encryption_key_id=1;
	Query OK, 2 rows affected (0.02 sec)               
	Records: 2  Duplicates: 0  Warnings: 0
	MariaDB [unenc]> show create table t;
	+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
	| Table | Create Table                                                                                                                                                     |
	+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
	| t     | CREATE TABLE `t` (
	  `id` int(11) DEFAULT NULL,
	  `value` varchar(255) DEFAULT NULL
	) ENGINE=InnoDB DEFAULT CHARSET=latin1 `encrypted`=yes `encryption_key_id`=1 |
	+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
	1 row in set (0.00 sec)
14. Run `strings` on t.ibd, its all encrpyted!
15. Undo it:
	MariaDB [unenc]> alter table t encrypted=no;
	Query OK, 2 rows affected (0.03 sec)               
	Records: 2  Duplicates: 0  Warnings: 0

	MariaDB [unenc]> select * from t;
	+------+----------+
	| id   | value    |
	+------+----------+
	|    1 | 12345    |
	|    2 | abc12345 |
	+------+----------+
	2 rows in set (0.00 sec)

	MariaDB [unenc]> \q
	Bye
	[root@centos-2gb-lon1-01 unenc]# strings t.ibd 
	infimum
	supremum
	12345
	abc12345
16. But what about the binlogs? `encrypt_binlog` in your config.

	MariaDB [(none)]> use unenc;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A

	Database changed
	MariaDB [unenc]> alter table t encrypted=yes encryption_key_id=1;
	Query OK, 2 rows affected (0.03 sec)               
	Records: 2  Duplicates: 0  Warnings: 0

	MariaDB [unenc]> insert into t values (3,'abc12345abc');
	Query OK, 1 row affected (0.00 sec)

	MariaDB [unenc]> select * from t;
	+------+-------------+
	| id   | value       |
	+------+-------------+
	|    1 | 12345       |
	|    2 | abc12345    |
	|    3 | abc12345abc |
	+------+-------------+
	3 rows in set (0.00 sec)

	ERROR: Error in Log_event::read_log_event(): 'Found invalid event in binary log', data_len: 39, event_type: 26
	/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
	/*!40019 SET @@session.max_insert_delayed_threads=0*/;
	/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
	DELIMITER /*!*/;
	# at 4
	#160407  6:12:33 server id 1  end_log_pos 249   Start: binlog v 4, server v 10.1.13-MariaDB created 160407  6:12:33 at startup
	# Warning: this binlog is either in use or was not closed properly.
	ROLLBACK/*!*/;
	BINLOG '
	kTIGVw8BAAAA9QAAAPkAAAABAAQAMTAuMS4xMy1NYXJpYURCAGxvZwAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAACRMgZXEzgNAAgAEgAEBAQEEgAA3QAEGggAAAAICAgCAAAACgoKAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAEEwQAAKGXC4I=
	'/*!*/;
	# at 249
	# Encryption scheme: 1, key_version: 1, nonce: 9c0ce381dcde9f8c5d522b45
	# The rest of the binlog is encrypted!
	DELIMITER ;
	# End of log file
	ROLLBACK /* added by mysqlbinlog */;
	/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
	/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

	MariaDB [unenc]> select * from t;
	ERROR 1296 (HY000): Got error 192 'Table encrypted but decryption failed. This could be because correct encryption management plugin is not loaded, used encryption key is not available or encryption method does not match.' from InnoDB
	
17. `show binary logs;` and then `show binlog events in 'centos-2gb-lon1-01-bin.000007';`. You may see an event like `| centos-2gb-lon1-01-bin.000005 | 563 | Annotate_rows     |         1 |         620 | insert into t values (3,'abc12345abc')                       |`, but even if you annotate rows, you will not be able to see anything in the binlog as seen above