# Installation on CentOS 7

1. Use the [repository configuration tool](https://downloads.mariadb.org/mariadb/repositories) to add to `/etc/yum.repos.d/` a file called `MariaDB.repo`. It is normal for it to replace the dated MariaDB Server 5.5 that comes as a default in CentOS 7.
> Installing:
>  MariaDB-shared      x86_64      10.1.13-1.el7.centos        mariadb      1.3 M
>      replacing  mariadb-libs.x86_64 1:5.5.47-1.el7_2
> Installing for dependencies:
>  MariaDB-common      x86_64      10.1.13-1.el7.centos        mariadb       43 k

2. `yum install mariadb-server mariadb-client`
3. 	`systemctl start mariadb` (note that you can't do something like `service status mariadb` or `/etc/init.d/mysql start`, since now everything has moved to `systemd`, and `systemctl` is the way to control it).

# Configuration on CentOS 7
You will notice that the `/etc/my.cnf` is pretty bare, encouraging you to discover `/etc/my.cnf.d/`. 

It is also worth noting that now in your `my.cnf`, you can have just MariaDB Server specific options using the `[mariadb]` group, and you can have version specific features, like the `[mariadb-10.1]` group (so if the same `my.cnf` is used on MariaDB Server 10.0, it will not be parsed).

# Percona Server 5.7
Note that the validate_password plugin is installed by default. You might need to read the [documentation](http://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html) to understand how these changes affect you.

	[root@centos-1gb-lon1-01 ~]# sudo grep 'temporary password' /var/log/mysqld.log
	2016-04-07T11:40:24.181384Z 1 [Note] A temporary password is generated for root@localhost: #=AHkyF(I7.1
	[root@centos-1gb-lon1-01 ~]# mysql -uroot -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 5
	Server version: 5.7.11-4

	Copyright (c) 2009-2016 Percona LLC and/or its affiliates
	Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.

	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

	mysql> 

	ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
	
