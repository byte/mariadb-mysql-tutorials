# Password validation

## Simple Password Check
This is achieved simply by doing: `install soname 'simple_password_check';`. It is [documented](https://mariadb.com/kb/en/mariadb/simple_password_check/).

Find out more by doing:
	SELECT VARIABLE_NAME, DEFAULT_VALUE, VARIABLE_COMMENT FROM INFORMATION_SCHEMA.SYSTEM_VARIABLES WHERE VARIABLE_NAME LIKE 'SIMPLE_PASSWORD%'\G
	*************************** 1. row ***************************
	   VARIABLE_NAME: SIMPLE_PASSWORD_CHECK_DIGITS
	   DEFAULT_VALUE: 1
	VARIABLE_COMMENT: Minimal required number of digits
	*************************** 2. row ***************************
	   VARIABLE_NAME: SIMPLE_PASSWORD_CHECK_LETTERS_SAME_CASE
	   DEFAULT_VALUE: 1
	VARIABLE_COMMENT: Minimal required number of letters of the same letter case.This limit is applied separately to upper-case and lower-case letters
	*************************** 3. row ***************************
	   VARIABLE_NAME: SIMPLE_PASSWORD_CHECK_OTHER_CHARACTERS
	   DEFAULT_VALUE: 1
	VARIABLE_COMMENT: Minimal required number of other (not letters or digits) characters
	*************************** 4. row ***************************
	   VARIABLE_NAME: SIMPLE_PASSWORD_CHECK_MINIMAL_LENGTH
	   DEFAULT_VALUE: 8
	VARIABLE_COMMENT: Minimal required password length
	4 rows in set (0.01 sec)

You will now try setting a simple password:
	MariaDB [(none)]> SET PASSWORD = PASSWORD('abc');
	ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

Or a more complex one that satisfies the requirements:
	MariaDB [(none)]> SET PASSWORD = PASSWORD('abc12345A%');
	Query OK, 0 rows affected (0.00 sec)

However, is that really a good password?

	echo "abc12345A%" |cracklib-check 
	abc12345A%: it is too simplistic/systematic

## Cracklib Password Check
On CentOS 7, you will need to first install the package by executing: `yum install MariaDB-cracklib-password-check`.

After that the process is similar:
	MariaDB [(none)]> INSTALL SONAME 'cracklib_password_check';
	Query OK, 0 rows affected (0.00 sec)

	MariaDB [(none)]> SELECT VARIABLE_NAME, DEFAULT_VALUE, VARIABLE_COMMENT FROM INFORMATION_SCHEMA.SYSTEM_VARIABLES WHERE VARIABLE_NAME LIKE 'CRACKLIB_PASSWORD%'\G
	*************************** 1. row ***************************
	   VARIABLE_NAME: CRACKLIB_PASSWORD_CHECK_DICTIONARY
	   DEFAULT_VALUE: /usr/share/cracklib/pw_dict
	VARIABLE_COMMENT: Path to a cracklib dictionary
	1 row in set (0.01 sec)

Now the password that was set previously does not work:
	MariaDB [(none)]> set password = password('abc12345A%');
	ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

	echo "LwBvmop9%oftB,x8"|cracklib-check 
	LwBvmop9%oftB,x8: OK

	MariaDB [(none)]> set password = password('LwBvmop9%oftB,x8');
	Query OK, 0 rows affected (0.00 sec)
