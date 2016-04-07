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
