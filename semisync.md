# Semisynchronous replication

Going beyond standard asynchronous replication that is provided in MySQL, one may also want to try semisynchronous replication. Semisynchronous replication implies that after a commit has occurred on a master, a wait will occur until one of the configured slaves has logged the event to disk.

Your server must have the ability to dynamically load plugins. Check this:

	show variables like 'have_dynamic_loading';
	+----------------------+-------+
	| Variable_name        | Value |
	+----------------------+-------+
	| have_dynamic_loading | YES   |
	+----------------------+-------+

It is important to note that there are two separate plugins for semisynchronous replication: one to be used on the master, and the other to be used on slaves.

Then (on the master):

	install plugin rpl_semi_sync_master soname 'semisync_master.so';
	Query OK, 0 rows affected (0.00 sec)
	
Verify: 

	SHOW PLUGINS;
	*************************** 45. row ***************************
	   Name: rpl_semi_sync_master
	 Status: ACTIVE
	   Type: REPLICATION
	Library: semisync_master.so
	License: GPL

On the slave:

	install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
	Query OK, 0 rows affected (0.00 sec)

Verify:
	
	SHOW PLUGINS;
	*************************** 45. row ***************************
	   Name: rpl_semi_sync_slave
	 Status: ACTIVE
	   Type: REPLICATION
	Library: semisync_slave.so
	License: GPL

The system variables to toggle the use of semisync replication are similar on the master and the slave. To enable ensure that `rpl_semi_sync_master_enabled` and `rpl_semi_sync_slave_enabled` have a value of one (1). Semisync is dynamic, so you can toggle this at runtime.

Is it on?

	SHOW GLOBAL VARIABLES LIKE 'rpl_semi_sync_master_enabled';
	+------------------------------+-------+
	| Variable_name                | Value |
	+------------------------------+-------+
	| rpl_semi_sync_master_enabled | OFF   |
	+------------------------------+-------+
	
No, so its time to turn it on.

	set global rpl_semi_sync_master_enabled=1;

Verify as above. Enable it on the slave as well: `set global rpl_semi_sync_slave_enabled=1;`. Then verify.

You can see this in the error log as well, to ensure that things are working:
2016-04-16T07:06:22.809205Z 8 [Note] Semi-sync replication initialized for transactions.
2016-04-16T07:06:22.809244Z 8 [Note] Semi-sync replication enabled on the master.
2016-04-16T07:06:22.809690Z 0 [Note] Starting ack receiver thread

One thing to remember when configuring semisync replication is that you need to set the master timeout via `rpl_semi_sync_master_timeout`; the default is set at 10000 which translates to 10 seconds. You might want to change this to 1 second (1000). Do this via: `set global rpl_semi_sync_master_timeout=1000;`. Verify via: `show variables like 'rpl_semi_sync_master_timeout';`

When you have decided to permanently use semisync replication in your topology, you can edit your `my.cnf` file to add:

	[mysqld]
	rpl_semi_sync_master_enabled = 1
	rpl_semi_sync_master_timeout = 1000
	
And on the slaves you would add:
	
	[mysqld]
	rpl_semi_sync_slave_enabled = 1
	
To monitor semisync replication, take a look at: `SHOW GLOBAL VARIABLES LIKE 'rpl_%';` and `SHOW GLOBAL STATUS like 'rpl_%';