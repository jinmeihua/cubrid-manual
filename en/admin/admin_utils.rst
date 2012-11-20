***********************
Database Administration
***********************
How to Use the CUBRID Management Utilities (Syntax)
===================================================

The following shows how to use the CUBRID management utilities. ::

	cubrid utility_name
	utility_name:
		createdb [option] <database_name>   --- Creating a database
		deletedb [option] <database_name>   --- Deleting a database
		installdb [option] <database-name>   --- Installing a database 
		renamedb [option] <source-database-name> <target-database-name>  --- Renaming a database 
	  	copydb [option] <source-database-name> <target-database-name>  --- Copying a database 
	  	backupdb [option] <database-name>  --- Backing up a database 
	  	restoredb [option] <database-name>  --- Restoring a database 
	  	addvoldb [option] <database-name>  --- Adding a database volume file 
	  	spacedb [option] <database-name>  --- Displaying details of database space 
	  	lockdb [option] <database-name>  --- Displaying details of database lock 
	  	killtran [option] <database-name>  --- Removing transactions 
	  	optimizedb [option] <database-name>  --- Updating database statistics 
	  	statdump [option] <database-name>  --- Outputting statistic information of database server execution 
	  	compactdb [option] <database-name>  --- Optimizing space by freeing unused space 
	  	diagdb [option] <database-name>  --- Displaying internal information 
	  	checkdb [option] <database-name>  --- Checking database consistency 
	  	alterdbhost [option] <database-name>  --- Altering database host 
	  	plandump [option] <database-name>  --- Displaying details of the query plan 
	  	loaddb [option] <database-name>  --- Loading data and schema 
	  	unloaddb [option] <database-name>  --- Unloading data and schema 
	 	paramdump [option] <database-name>  --- Checking out the parameter values configured in a database 
	  	changemode [option] <database-name>  --- Displaying or changing the server HA mode 
	  	copylogdb [option] <database-name>  --- Multiplating transaction logs to configure HA 
	  	applylogdb [option] <database-name>  --- Reading and applying replication logs from transaction logs to configure HA 
	
Database Users
==============

A CUBRID database user can have members with the same authorization. If authorization **A** is granted to a user, the same authorization is also granted to all members belonging to the user. A database user and its members are called a "group."; a use who has no members is called a "user."

CUBRID provides **DBA** and **PUBLIC** users by default.

* **DBA** can access every object in the database, that is, it has authorization at the highest level. Only **DBA** has sufficient authorization to add, alter and delete the database users.

* All users including **DBA** are members of **PUBLIC**. Therefore, all database users have the authorization granted to **PUBLIC** . For example, if authorization **B** is added to **PUBLIC** group, all database members will automatically have the **B** authorization.

.. _databases-txt-file:

databases.txt File
==================

CUBRID stores information on the locations of all existing databases in the **databases.txt** file. This file is called the "database location file." A database location file is used when CUBRID executes utilities for creating, renaming, deleting or replicating databases; it is also used when CUBRID runs each database. By default, this file is located in the **databases** directory under the installation directory. The directory is located through the environment variable **CUBRID_DATABASES**. 

::

	db_name db_directory server_host logfile_directory

The format of each line of a database location file is the same as defined by the above syntax; it contains information on the database name, database path, server host and the path to the log files. The following example shows how to check the contents of a database location file.

::

	% more databases.txt

	dist_testdb /home1/user/CUBRID/bin d85007 /home1/user/CUBRID/bin
	dist_demodb /home1/user/CUBRID/bin d85007 /home1/user/CUBRID/bin
	testdb /home1/user/CUBRID/databases/testdb d85007 /home1/user/CUBRID/databases/testdb
	demodb /home1/user/CUBRID/databases/demodb d85007 /home1/user/CUBRID/databases/demodb

By default, the database location file is stored in the **databases**
directory under the installation directory. You can change the default directory by modifying the value of the **CUBRID_DATABASES** environment variable. The path to  the database location file must be valid so that the **cubrid** utility for database management can access the file properly. You must enter the directory path correctly and check if you have write permission on the file. The following example shows how to check the value configured in the **CUBRID_DATABASES** environment variable.

::

	% set | grep CUBRID_DATABASES
	CUBRID_DATABASES=/home1/user/CUBRID/databases

An error occurs if an invalid directory path is set in the **CUBRID_DATABASES** environment variable. If the directory path is valid but the database location file does not exist, a new location information file is created. If the **CUBRID_DATABASES** environment variable has not been configured at all, CUBRID retrieves the location information file in the current working directory.

.. _creating-database:

Creating Database
=================

The **cubrid createdb** utility creates databases and initializes them with the built-in CUBRID system tables. It can also define initial users to be authorized in the database and specify the locations of the logs and databases. In general, the **cubrid createdb** utility is used only by DBA. 

::

	cubrid createdb <options> database_name

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **createdb**: A command used to create a new database.

* *database_name*: Specifies a unique name for the database to be created, without including the path name to the directory where the database will be created. If the specified database name is the same as that of an existing database name, CUBRID halts creation of the database to protect existing files.

The following shows options available with the **cubrid** **createdb** utility.

.. program:: createdb

.. option:: --db-volume-size=SIZE

	This option specifies the size of the database volume that will be created first. The default value is  the value of the system parameter
	**db_volume_size**, and the minimum value is 20M. You can set units as K, M, G and T, which stand for kilobytes (KB), megabytes (MB), gigabytes (GB), and terabytes (TB) respectively. If you omit the unit, bytes will be applied.

	The following example shows how to create a database named *testdb* and assign 512 MB to its first volume. ::
	
		cubrid createdb --db-volume-size=512M testdb

.. option:: --db-page-size=SIZE

	This option specifies the size of the database page; the minimum value is 4K and the maximum value is
	**16K** (default). K stands for kilobytes (KB).

	The value of page size is one of the followings: 4K, 8K, or 16K. If a value between 4K and 16K is specified, system rounds up the number. If a value greater than 16K or less than 4K, the specified number is used.

	The following example shows how to create a database named *testdb* and configure its page size 16K. ::

		cubrid createdb --db-page-size=16K testdb

.. option:: --log-volume-size=SIZE

	This option  specifies the size of the database log volume. The default value is the same as database volume size, and the minimum value is 20M. You can set units as K, M, G and T, which stand for kilobytes (KB), megabytes (MB), gigabytes (GB), and terabytes (TB) respectively. If you omit the unit, bytes will be applied. 

	The following example shows how to create a database named *testdb* and assign 256 MB to its log volume. ::

		cubrid createdb --log-volume-size=256M testdb

.. option:: --log-page-size=SIZE

	This option specifies the size of the log volume page. The default value is the same as data page size. The minimum value is 4K and the maximum value is 16K. K stands for kilobytes (KB).

	The value of page size is one of the followings: 4K, 8K, or 16K. If a value between 4K and 16K is specified, system rounds up the number. If a value greater than 16K or less than 4K, the specified number is used.

	The following example shows how to create  a database named *testdb* and configure its log volume page size 8K. ::

		cubrid createdb --log-page-size=8K testdb

.. option:: --comment=COMMENT

	This option specifies a comment to be included in the database volume header. If the character string contains spaces, the comment must be enclosed in double quotes.

	The following example shows how to create a database named *testdb* and add a comment to the database volume. ::

		cubrid createdb --comment "a new database for study" testdb

.. option:: -F, --file_path=PATH

	The **-F** option specifies an absolute path to a directory where the new database will be created. If the **-F** option is not specified, the new database is created in the current working directory.
	
	The following example shows how to create a database named *testdb* in the directory /dbtemp/new_db. ::
		cubrid createdb -F "/dbtemp/new_db/" testdb

.. option:: -L log_path=PATH

	The **-L** option specifies an absolute path to the directory where database log files are created. If the **-L** option is not specified, log files are created in the directory specified by the **-F** option. If neither **-F** nor **-L** option is specified, database log files are created in the current working directory.

	The following example shows how to create a database named *testdb* in the directory /dbtemp/newdb and log files in the directory /dbtemp/db_log. ::

		cubrid createdb -F "/dbtemp/new_db/" -L "/dbtemp/db_log/" testdb

.. option:: -B, --lob-base-path=PATH

	This option specifies a directory where LOB data files are stored when BLOB/CLOB data is used. If the **--lob-base-path** option is not specified, LOB data files are store in < *location of database volumns created* >/ **lob** directory. The following example shows how to create a database named *testdb* in the working directory and specify /home/data1 of local file system as a location of LOB data files. ::

		cubrid createdb --lob-base-path "file:/home1/data1" testdb
		
.. option:: --server-name=HOST

	This option enables the server of a specific database to run in the specified host when CUBRID client/server is used. The information of a host specified is stored in the **databases.txt** file. If this option is not specified, the current localhost is specified by default. The following example shows how to create a database named *testdb* and register it on the host *aa_host*. ::

		cubrid createdb --server-name aa_host testdb

.. option:: -r

	This option creates a new database and overwrites an existing database if one with the same name exists. If the **-r** option is not specified, database creation is halted. The following example shows how to create a new database named *testdb* and overwrite the existing database with the same name. ::
	
		cubrid createdb -r testdb
	
.. option:: --more-volume-file=FILE

	This option creates an additional volume based on the specification contained in the file specified by the option. The volume is created in the same directory where the database is created. Instead of using this option, you can add a volume by using the **cubrid addvoldb** utility. The following example shows how to create a database named *testdb* as well as an additional volume based on the specification stored in the **vol_info.txt** file. ::

		cubrid createdb --more-volume-file vol_info.txt testdb

	The following is a specification of the additional volume contained in the **vol_info.txt** file. The specification of each volume must be written on a single line. ::

		#xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
		# NAME volname COMMENTS volcmnts PURPOSE volpurp NPAGES volnpgs
		NAME data_v1 COMMENTS "data information volume" PURPOSE data NPAGES 1000
		NAME data_v2 COMMENTS "data information volume" PURPOSE data NPAGES 1000
		NAME data_v3 PURPOSE data NPAGES 1000
		NAME index_v1 COMMENTS "index information volume" PURPOSE index NPAGES 500
		NAME temp_v1 COMMENTS "temporary information volume" PURPOSE temp NPAGES 500
		NAME generic_v1 COMMENTS "generic information volume" PURPOSE generic NPAGES 500
		#xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

	As shown in the example, the specification of each volume consists followings. ::

		NAME volname COMMENTS volcmnts PURPOSE volpurp NPAGES volnpgs

	* *volname*: The name of the volume to be created. It must follow the UNIX file name conventions and be a simple name not including the directory path. The specification of a volume name can be omitted. If it is, the "database name to be created by the system_volume identifier" becomes the volume name.

	* *volcmnts*: Comment to be written in the volume header. It contains information on the additional volume to be created. The specification of the comment on a volume can also be omitted.

	* *volpurp*: It must be one of the following types: **data**, **index**, **temp**, or **generic** based on the purpose of storing volumes. The specification of the purpose of a volume can be omitted in which case the default value is **generic**.

	* *volnpgs*: The number of pages of the additional volume to be created. The specification of the number of pages of the volume cannot be omitted; it must be specified.

.. option:: --user-definition-file=FILE

	This option adds users who have access to the database to be created. It adds a user based on the specification contained in the user information file specified by the parameter. Instead of using the **--user-definition-file** option, you can add a user by using the **CREATE USER** statement (for details, see `Managing USER <#syntax_syntax_access_manage_htm>`_ ).

	The following example shows how to create a database named *testdb* and add users to *testdb* based on the user information defined in the **user_info.txt** file. ::

		cubrid createdb --user-definition-file user_info.txt testdb

	The syntax of a user information file is as follows: ::

		USER user_name [ <groups_clause> | <members_clause> ]
		
		<groups_clause>: 
			[ GROUPS <group_name> [ { <group_name> }... ] ]

		<members_clause>: 
			[ MEMBERS <member_name> [ { <member_name> }... ] ]

	* The *user_name* is the name of the user who has access to the database. It must not include spaces.

	* The **GROUPS** clause is optional. The *group_name* is the upper level group that contains the *user_name* . Here, the *group_name* can be multiply specified and must be defined as **USER** in advance.

	* The **MEMBERS** clause is optional. The *member_name* is the name of the lower level member that belongs to the *user_name* . Here, the *member_name* can be multiply specified and must be defined as **USER** in advance.

	Comments can be used in a user information file. A comment line must begin with a consecutive hyphen lines (--). Blank lines are ignored.

	The following example shows a user information in which *grandeur* and *sonata* are included in *sedan* group, *tuscan* is included in *suv* group, and *i30* is included in *hatchback* group. The name of the user information file is **user_info.txt**. ::

	
		--
		-- Example 1 of a user information file
		--
		USER sedan
		USER suv
		USER hatchback
		USER granduer GROUPS sedan
		USER sonata GROUPS sedan
		USER tuscan GROUPS suv
		USER i30 GROUPS hatchback

	The following example shows a file that has the same user relationship information as the file above. The difference is that the **MEMBERS** statement is used in the file below. ::

		--
		-- Example 2 of a user information file
		--
		USER granduer
		USER sonata
		USER tuscan
		USER i30
		USER sedan MEMBERS sonata granduer
		USER suv MEMBERS tuscan
		USER hatchback MEMBERS i30
	
.. option:: --csql-initialization-file=FILE

	This option executes an SQL statement on the database to be created by using the CSQL Interpreter. A schema can be created based on the SQL statement contained in the file specified by the parameter.

	The following example shows how to create a database named *testdb* and execute the SQL statement defined in table_schema.sql through the CSQL Interpreter. ::

		cubrid createdb --csql-initialization-file table_schema.sql testdb

.. option:: -o

	This option stores messages related to the database creation to the file given as a parameter. The file is created in the same directory where the database was created. If the **-o** option is not specified, messages are displayed on the console screen. The **-o** option allows you to use information on the creation of a certain database by storing messages, generated during the database creation, to a specified file.

	The following example shows how to create a database named *testdb* and store the output of the utility to the **db_output** file instead of displaying it on the console screen. ::

		cubrid createdb -o db_output testdb

.. option:: -v

	This option displays all information on the database creation operation onto the screen. Like the **-o** option, this option is useful in checking information related to the creation of a specific database. Therefore, if you specify the **-v** option together with the **-o** option, you can store the output messages in the file given as a parameter; the messages contain the operation information about the **cubrid createdb** utility and database creation process.

	The following example shows how to create a database named *testdb* and display detailed information on the operation onto the screen. ::

		cubrid createdb -v testdb

.. note::

	**temp_file_max_size_in_pages** is a parameter used to configure the maximum number of pages assigned to store the temporary temp volume - used for complicated queries or storing arrays - on the disk. 

	While the default value is **-1**, the temporary temp volume may be increased up to the amount of extra space on the disk specified by the
	**temp_volume_path** parameter. If the value is 0, the temporary temp volume cannot be created. In this case, the permanent temp volume should be added by using the `cubrid addvoldb <#admin_admin_db_addvol_htm>`_ utility.

	For the efficient management of the volume, it is recommended to add a volume for each usage. By using the `cubrid spacedb <#admin_admin_db_space_htm>`_
	utility, you can check the reaming space of each volume. By using the `cubrid addvoldb <#admin_admin_db_addvol_htm>`_ utility, you can add more volumes as needed while managing the database. When adding a volume while managing the database, you are advised to do so when there is less system load. Once the assigned volume for a usage is completely in use, a generic volume will be created, so it is suggested to add extra volume for a usage that is expected to require more space.

	Next, we will look at how to add volumes for **data**, **index**, and **temp** by creating the database and separating the volume usage. ::

		cubrid createdb --db-volume-size=512M --log-volume-size=256M cubriddb
		cubrid addvoldb -p data -n cubriddb_DATA01 --db-volume-size=512M cubriddb
		cubrid addvoldb -p data -n cubriddb_DATA02 --db-volume-size=512M cubriddb
		cubrid addvoldb -p index -n cubriddb_INDEX01 cubriddb --db-volume-size=512M cubriddb
		cubrid addvoldb -p temp -n cubriddb_TEMP01 cubriddb --db-volume-size=512M cubriddb

.. _adding-database-volume:

Adding Database Volume
======================

Adds database volume. ::

	cubrid addvoldb [options] database_name

* **cubrid**: An integrated utility for CUBRID service and database management.

* **addvoldb**: A command that adds a specified number of pages of the new volume to a specified database.

* *database_name*: Specifies the name of the database to which a volume is to be added without including the path name to the directory where the database is to be created.

The following shows options available with the **cubrid addvoldb** utility.

.. program:: addvoldb

.. option:: --db-volume-size=SIZE

	**--db-volume-size** is an option that specifies the size of the volume to be added to a specified database. If the **--db-volume-size** option is omitted, the value of the system parameter **db_volume_size** is used by default. You can set units as K, M, G and T, which stand for kilobytes (KB), megabytes (MB), gigabytes (GB), and terabytes (TB) respectively. If you omit the unit, bytes will be applied.

	The following example shows how to add a volume for which 256 MB are assigned to the *testdb* database. ::

		cubrid addvoldb -p data --db-volume-size=256M testdb

.. option:: -n NAME

	This option specifies the name of the volume to be added to a specified database. The volume name must follow the file name protocol of the operating system and be a simple one without including the directory path or spaces. If the **-n** option is omitted, the name of the volume to be added is configured by the system automatically as "database name_volume identifier." For example, if the database name is *testdb*, the volume name *testdb_x001* is automatically configured.
	
	The following example shows how to add a volume for which 256 MB are assigned to the *testdb* database in standalone mode. The volume name *testdb_v1* will be created. ::

		cubrid addvoldb -S -n testdb_v1 --db-volume-size=256M testdb

.. option::  -F, --file-path=PATH

	This option specifies the directory path where the volume to be added will be stored. If the **-F** option is omitted, the value of the system parameter **volume_extension_path** is used by default.

	The following example shows how to add a volume for which 256 MB are assigned to the *testdb* database in standalone mode. The added volume is created in the /dbtemp/addvol directory. Because the **-n** option is not specified for the volume name, the volume name *testdb_x001* will be created. ::

		cubrid addvoldb -S -F /dbtemp/addvol/ --db-volume-size=256M testdb

.. option:: --comment COMMENT

	This option facilitates to retrieve information on the added volume by adding such information in the form of comments. It is recommended that the contents of a comment include the name of **DBA** who adds the volume, or the purpose of adding the volume. The comment must be enclosed in double quotes.  The following example shows how to add a volume for which 256 MB are assigned to the *testdb* database in standalone mode and inserts a comment about the volume. ::

		cubrid addvoldb -S --comment "data volume added_cheolsoo kim" --db-volume-size=256M testdb

.. option:: -p PURPOSE

	This option specifies the purpose of the volume to be added. The reason for specifying the purpose of the volume is to improve the I/O performance by storing volumes separately on different disk drives according to their purpose. Parameter values that can be used for the **-p** option are **data**, **index**, **temp** and **generic**. The default value is **generic**. For the purpose of each volume, see "`Database Volume Structure <#intro_intro_arch_volume_htm>`_ ."

	The following example shows how to add a volume for which 256 MB are assigned to the *testdb* database in standalone mode. ::
	
		cubrid addvoldb -S -p index --db-volume-size=256M testdb

.. option::  -S, --SA-mode

	This option accesses the database in standalone mode without running the server process. This option has no parameter. If the **-S** option is not specified, the system assumes to be in client/server mode. ::

		cubrid addvoldb -S --db-volume-size=256M testdb

.. option::  -C, --CS-mode

	This option accesses the database in client/server mode by running the server and the client separately. There is no parameter. Even when the **-C** option is not specified, the system assumes to be in client/server mode by default. ::

		cubrid addvoldb -C --db-volume-size=256M testdb

The following example shows how to create a database, classify volume usage, and add volumes such as **data**, **index**, and **temp**. ::

	cubrid createdb --db-volume-size=512M --log-volume-size=256M cubriddb
	cubrid addvoldb -p data -n cubriddb_DATA01 --db-volume-size=512M cubriddb
	cubrid addvoldb -p data -n cubriddb_DATA02 --db-volume-size=512M cubriddb
	cubrid addvoldb -p index -n cubriddb_INDEX01 cubriddb --db-volume-size=512M cubriddb
	cubrid addvoldb -p temp -n cubriddb_TEMP01 cubriddb --db-volume-size=512M cubriddb

Deleting Database
=================

The **cubrid deletedb** utility is used to delete a database. You must use the **cubrid deletedb** utility to delete a database, instead of using the file deletion commands of the operating system; a database consists of a few interdependent files. The **cubrid deletedb** utility also deletes the information on the database from the database location file (**databases.txt**). The **cubrid deletedb** utility must be run offline, that is, in standalone mode when nobody is using the database.

cubrid deletedb  [options] database_name

	* **cubrid**: An integrated utility for the CUBRID service and database management.

	* **deletedb**: A command to delete a database, its related data, logs and all backup files. It can be executed successfully only when the database is in a stopped state.

	* *database_name*: Specifies the name of the database to be deleted without including the path name.

The following shows options available with the **cubrid deleteldb** utility.
	
.. program:: deletedb
	
.. option:: -o, --output-file=FILE

	This option specifies the file name for writing messages::

		cubrid deletedb -o deleted_db.out testdb

	The **cubrid** **deletedb** utility also deletes the database information contained in the database location file (**databases.txt**). The following message is returned if you enter a utility that tries to delete a non-existing database. ::

		cubrid deletedb testdb
		
		Database "testdb" is unknown, or the file "databases.txt" cannot be accessed.

.. option:: -d, --delete-backup

	This option deletes database volumns, backup volumes and backup information files simultaneously. If the -**d** option is not specified, backup volume and backup information files are not deleted. ::
	
		cubrid deletedb -d testdb

Renaming Database
=================

The **cubrid renamedb** utility renames a database. The names of information volumes, log volumes and control files are also renamed to conform to the new database one.

In contrast, the **cubrid alterdbhost** utility configures or changes the host name of the specified database. In other words, it changes the host name configuration in the **databases.txt** file. ::

	cubrid renamedb [options] src_database_name dest_database_name

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **renamedb**: A command that changes the existing name of a database to a new one. It executes successfully only when the database is in a stopped state. The names of related information volumes, log volumes and control files are also changed to new ones accordingly.

* *src_database_name*: The name of the existing database to be renamed. The path name to the directory where the database is to be created must not be included.

* *dest_database_name*: The new name of the database. It must not be the same as that of an existing database. The path name to the directory where the database is to be created must not be included.

The following shows [options] available with the **cubrid deleteldb** utility.
	 
.. program:: alterdbhost

.. option:: -E, --extended-volume-path=PATH

	This option renames an extended volume created in a specific directory path (e.g. /dbtemp/addvol/), and then moves the volume to a new directory. This specifies a new directory path (e.g. /dbtemp/newaddvols/) where the renamed extended volume will be moved. If it is not specified, the extended volume is only renamed in the existing path without being moved. If a directory path outside the disk partition of the existing database volume or an invalid one is specified, the rename operation is not executed. This option cannot be used together with the **-i** option. ::

		cubrid renamedb -E /dbtemp/newaddvols/ testdb testdb_1

.. option:: -i, --control-file=FILE

	The option specifies an input file in which directory information is stored to change all database name of volumes or files and assign different directory at once. To perform this work, the **-i** option is used. The **-i** option cannot be used together with the **-E** option. ::
	
		cubrid renamedb -i rename_path testdb testdb_1

	The followings are the syntax and example of a file that contains the name of each volume, the current directory path and the directory path where renamed volumes will be stored. ::

		volid source_fullvolname dest_fullvolname

	* *volid*: An integer that is used to identify each volume. It can be checked in the database volume control file (database_name_vinf).

	* *source_fullvolname*: The current directory path to each volume.

	* *dest_fullvolname*: The target directory path where renamed volumes will be moved. If the target directory path is invalid, the database rename operation is not executed.

	::

		-5  /home1/user/testdb_vinf       /home1/CUBRID/databases/testdb_1_vinf
		-4  /home1/user/testdb_lginf      /home1/CUBRID/databases/testdb_1_lginf
		-3  /home1/user/testdb_bkvinf     /home1/CUBRID/databases/testdb_1_bkvinf
		-2  /home1/user/testdb_lgat       /home1/CUBRID/databases/testdb_1_lgat
		 0  /home1/user/testdb            /home1/CUBRID/databases/testdb_1
		 1  /home1/user/backup/testdb_x001/home1/CUBRID/databases/backup/testdb_1_x001

.. option:: -d, --delete-backup

	This option renames the *testdb* database and at once forcefully delete all backup volumes and backup information files that are in the same location as *testdb*. Note that you cannot use the backup files with the old names once the database is renamed. If the **-d** option is not specified, backup volumes and backup information files are not deleted. ::
	
		cubrid renamedb -d testdb testdb_1

Renaming Database Host
======================

The **cubrid alterdbhost** utility sets or changes the host name of the specified database. It changes the host name set in the **databases.txt** file. ::

	cubrid alterdbhost [option] database_name
	
* **cubrid**: An integrated utility for the CUBRID service and database management

* **alterdbhost**: A command used to change the host name of the current database

.. program:: alterdbhost

.. option:: -h, --host=HOST

	The *-h* option specifies the host name to be changed. When this option is omitted, specifies the host name to localhost.

Copying/Moving Database
=======================

The **cubrid copydb** utility copy or move a database to another location. As arguments, source and target name of database must be given. A target database name must be different from a source database name. When the target name argument is specified, the location of target database name is registered in the **databases.txt**
file. The **cubrid copydb** utility can be executed only offline (that is, state of a source database stop). ::

	cubrid copydb [options] src-database-name dest-database-name

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **copydb**: A command that copy or move a database from one to another location.

* *src-database-name*: The names of source and target databases to be copied or moved.

* *dest-database-name*: A new (target) database name.

If options are omitted, a target database is copied into the same directory of a source database.

The following shows [options] available with the **cubrid copydb** utility.

.. program:: copydb

.. option:: --server-name=HOST

	The *--server-name* option specifies a host name of new database. The host name is registered in the **databases.txt** file. If this option is omitted, a local host is registered. ::
	
		cubrid copydb --server-name=cub_server1 demodb new_demodb

.. option:: -F, --file-path=PATH

	The *-F* option specifies a specific directory path where a new database volume is stored with an **-F** option. It represents specifying an absolute path. If the specified directory does not exist, an error is displayed. If this option is omitted, a new database volume is created in the current working directory. And this information is specified in **vol-path** of the **databases.txt** file. ::

		cubrid copydb -F /home/usr/CUBRID/databases demodb new_demodb

.. option:: -L, --log-path=PATH

	The *-L* option specifies a specific directory path where a new database volume is stored with an **-L** option. It represents specifying an absolute path. If the specified directory does not exist, an error is displayed. If this option is omitted, a new database volume is created in the current working directory. And this information is specified in **log-path** of the **databases.txt** file. ::

		cubrid copydb -L /home/usr/CUBRID/databases/logs demodb new_demodb

.. option:: -E, --extended-volume-path=PATH

	The *-E* option specifies a specific directory path where a new database extended volume is stored with an **-E**. If this option is omitted, a new database extended volume is created in the location of a new database volume or in the registered path of controlling file. The **-i** option cannot be used with this option. ::

		cubrid copydb -E home/usr/CUBRID/databases/extvols demodb new_demodb

.. option:: -i, --control_file=FILE

	The **-i** option specifies an input file where a new directory path information and a source volume are stored to copy or move multiple volumes into a different directory, respectively. This option cannot be used with the **-E** option. An input file named copy_path is specified in the example below. ::

		cubrid copydb -i copy_path demodb new_demodb

	The following is an exmaple of input file that contains each volume name, current directory path, and new directory and volume names. ::

		# volid   source_fullvolname   dest_fullvolname
		0 /usr/databases/demodb        /drive1/usr/databases/new_demodb
		1 /usr/databases/demodb_data1  /drive1/usr/databases/new_demodb new_data1
		2 /usr/databases/ext/demodb index1 /drive2//usr/databases/new_demodb new_index1
		3 /usr/ databases/ext/demodb index2  /drive2/usr/databases/new_demodb new_index2

	* *volid*: An integer that is used to identify each volume. It can be checked in the database volume control file (**database_name_vinf**).

	* *source_fullvolname*: The current directory path to each source database volume.

	* *dest_fullvolname*: The target directory path where new volumes will be stored. You should specify a vaild path.  

.. option:: -r, --replace

	If the **-r** option is specified, a new database name overwrites the existing database name if it is identical, insteading outputting an error. ::

		cubrid copydb -r -F /home/usr/CUBRID/databases demodb new_demodb

.. option:: -d, --delete-source

	If the **-d** option is specified, a source database is deleted after the database is copied. This execution brings the same the result as executing **cubrid deletedb** utility after copying a database. Note that if a source database contains LOB data, LOB file directory path of a source database is copied into a new database and it is registered in the **lob-base-path** of the **databases.txt** file. ::

		cubrid copydb -d -opyhome/usr/CUBRID/databases demodb new_demodb

.. option:: --copy-lob-path=PATH

	If the **--copy-lob-path** option is specified, a new directory path for LOB files is created and a source database is copied into a new directory path. If this option is omitted, the directory path is not created. Therefore, the **lob-base-path** of the **databases.txt** file should be modified separately. This option cannot be used with the **-B** option. ::

		cubrid copydb --copy-lob-path demodb new_demodb

.. option:: -B, --lob-base-path=PATH

	If the **-B** option is specified, a specified directory is specified as for LOB files of a new database and a source database is copied. This option cannot be used with the **--copy-lob-path** option. ::

		cubrid copydb -B /home/usr/CUBRID/databases/new_lob demodb new_demodb

Registering Database
====================

The **cubrid installdb** utility is used to register the information of a newly installed database to **databases.txt**, which stores database location information. The execution of this utility does not affect the operation of the database to be registered. ::

	cubrid installdb [options] database_name 
	
* **cubrid**: An integrated utility for the CUBRID service and database management.

* **installdb**: A command that registers the information of a moved or copied database to **databases.txt**.

* *database_name*: The name of database to be registered to **databases.txt**.

If no option is used with a command, the command must be executed in the directory where the corresponding database exists.

The following shows [options] available with the **cubrid installdb** utility.

.. program:: installdb

.. option:: --server-name=HOST

	This option registers the server host information of a database to **databases.txt** with a specific host name. If this is not specified, the current host information is registered. ::

		cubrid installdb --server-name=cub_server1 testdb

.. option::-F, --file-path=PATH

	This option registers the directory path of a database volume to **databases.txt** by using the **-F** option. If this option is not specified, the path of a current directory is registered as default. ::

		cubrid installdb -F /home/cubrid/CUBRID/databases/testdb testdb

.. option:: -L, --log-path=PATH

	This option registers the directory path of a database log volume to **databases.txt** by using the **-L** option. If this option is not specified, the directory path of a volume is registered. ::

		cubrid installdb -L /home/cubrid/CUBRID/databases/logs/testdb testdb

Checking Used Space
===================

The **cubrid spacedb** utility is used to check how much space of database volumes is being used. It shows a brief description of all permanent data volumes in the database. Information returned by the **cubrid spacedb** utility includes the ID, name, purpose and total/free space of each volume. You can also check the total number of volumes and used/unused database pages. ::

	cubrid spacedb [options] database_name

*  **cubrid** : An integrated utility for the CUBRID service and database management.

*  **spacedb** : A command that checks the space in the database. It executes successfully only when the database is in a stopped state.

*  *database_name* : The name of the database whose space is to be checked. The path-name to the directory where the database is to be created must not be included.

The following shows [options] available with the **cubrid spacedb** utility.
 
.. program:: spacedb

.. option:: -o FILE

	This option stores the result of checking the space information of *testdb* to a file named *db_output*. ::

		cubrid spacedb -o db_output testdb

.. option:: -S, --SA-mode

	This option is used to access a database in standalone, which means it works without processing server; it does not have an argument. If **-S** is not specified, the system recognizes that a database is running in client/server mode. ::

		cubrid spacedb --SA-mode testdb

.. option:: -C, --CS-mode

	This option is used to access a database in client/server mode, which means it works in client/server process respectively; it does not have an argument. If **-C** is not specified, the system recognize that a database is running in client/server mode by default. ::

		cubrid spacedb --CS-mode testdb

.. option:: --size-unit={PAGE|M|G|T|H}

	This option specifies the size unit of the space information of the database to be one of PAGE, M(MB), G(GB), T(TB), H(print-friendly). The default value is **H**. If you set the value to H, the unit is automatically determined as follows: M if 1 MB = DB size < 1024 MB, G if 1 GB = DB size < 1024 GB. ::
	
		cubrid spacedb --size_unit=M testdb
		cubrid spacedb --size_unit=H testdb

.. option:: -s, --summarize

	This option aggregates total_pages, used_pages and free_pages by DATA, INDEX, GENERIC, TEMP and TEMP TEMP, and outputs it. ::

		cubrid spacedb –s testdb

Compacting Used Space
=====================

The **cubrid compactdb** utility is used to secure unused space of the database volume. In case the database server is not running (offline), you can perform the job in standalone mode. In case the database server is running, you can perform it in client-server mode.

The **cubrid compactdb** utility secures the space being taken by OIDs of deleted objects and by class changes. When an object is deleted, the space taken by its OID is not immediately freed because there might be other objects that refer to the deleted one. Reference to the object deleted during compacting is displayed as **NULL**
, which means this can be reused by OIDs.

::

	cubrid compactdb [options] database_name [class_name], class_name2, ...]

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **compactdb**: A command that compacts the space of the database so that OIDs assigned to deleted data can be reused.

* *database_name*: The name of the database whose space is to be compacted. The path name to the directory where the database is to be created must not be included.

* *class_name_list*: You can specify the list of tables names that you want to compact space after a database name; the -i option cannot be used together. It is used in client/server mode only.

**-I**, **-i**, **-c**, **-d**, **-p** options are applied in client/server mode only.

The following shows [options] available with the **cubrid spacedb** utility.

.. program:: compactdb

.. option:: -v

	You can output messages that shows which class is currently being compacted and how many instances have been processed for the class by using the **-v** option. ::

		cubrid compactdb -v testdb

.. option:: -S, --SA-mode

	This option specifies to compact used space in standalone mode while database server is not running; no arugment is specified.  If the **-S** option is not specified, system recognizes that the job is executed in client/server mode. ::

		cubrid compactdb --SA-mode testdb

.. option:: C, --CS-mode

	This option specifies to compact used space in client/server mode while database server is running; no argument is specified. Even though this option is omitted, system recognizes that the job is executed in client/server mode. The following options can be used in client/server mode only.

.. option:: - i, --input-class-file=FILE

	You can specify an input file name that contains the table table name with this option. Write one table name in a single line; invalid table name is ignored. Note that you cannot specify the list of the table names after a database name in case of you use this option.

.. option:: -p, --pages-commited-once=NUMBER

	You can specify the number of maximum pages that can be commited once with this option. The default value is 10, the minimum value is 1, and the maximum value is 10. The less option value is specified, the more concurrency is enhanced because the value for class/instance lock is small; however, it causes slowdown on operation, and vice versa.

.. option:: -d, --delete-old-repr

	You can delete an existing table representation (schema structure) from catalog with this option. When a column is added or deleted by the **ALTER** statement, if the existing record still refers to the previous schema, no additional cost to update the schema is required and the previous table is kept.

.. option:: -I, --Instance-lock-timeout

	You can specify a value of instance lock timeout with this option. The default value is 2 (seconds), the minimum value is 1, and the maximum value is 10. The less option value is specified, the more operation speeds up. However, the number of instances that can be processed becomes smaller, and vice versa.

.. option:: -c, --class-lock-timeout

	You can specify a value of instance lock timeout with this option. The default value is 10 (seconds), the minimum value is 1, and the maximum value is 10. The less option value is specified, the more operation speeds up. However, the number of tables that can be processed becomes smaller, and vice versa. ::

		cubrid compactdb --CS-mode -p 10 testdb tbl1, tbl2, tbl5

Updating Statistics
===================

Updates statistical information such as the number of objects, the number of pages to access, and the distribution of attribute values. ::

	cubrid optimizedb [option] database_name

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **optimizedb**: Updates the statistics information, which is used for cost-based query optimization of the database. If the option is specified, only the information of the specified class is updated.

* *database_name*: The name of the database whose cost-based query optimization statistics are to be updated.

The following example shows how to update the query statistics information of all classes in the database. ::

	cubrid optimizedb testdb

The following shows [option] available with the **cubrid optimizedb** utility.
		
.. program :: optimizedb

.. option:: -n, --class-name

The following example shows how to update the query statistics information of the given class by using the **-n** option. ::

	cubrid optimizedb -n event_table testdb

.. _statdump:

Outputting Statistics Information of Server
===========================================

The cubrid statdump utility checks statistics information processed by the CUBRID database server. The statistics information mainly consists of the followings: File I/O, Page buffer, Logs, Transactions, Concurrency/Lock, Index, and Network request

Note that you must specify the parameter **communication_histogram** to **yes** in the **cubrid.conf** before executing the utility. You can also check statistics information of server with session commands (**;.h on**) in the CSQL.

::

	cubrid statdump [options] database_name

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **installdb**: A command that dumps the statistics information on the database server execution.

* *database_name*: The name of database which has the statistics data to be dumped.

The following shows [options] available with the **cubrid statdump** utility.

.. program:: statdump

.. option:: -i, --interval=SECOND

	The **-i** option specifies the periodic number of outputting statistics as seconds.

	::

		cubrid statdump -i 5 testdb
		 
		Thu April 07 23:10:08 KST 2011
		 
		 *** SERVER EXECUTION STATISTICS ***
		Num_file_creates              =          0
		Num_file_removes              =          0
		Num_file_ioreads              =          0
		Num_file_iowrites             =          0
		Num_file_iosynches            =          0
		Num_data_page_fetches         =          0
		Num_data_page_dirties         =          0
		Num_data_page_ioreads         =          0
		Num_data_page_iowrites        =          0
		Num_data_page_victims         =          0
		Num_data_page_iowrites_for_replacement =          0
		Num_log_page_ioreads          =          0
		Num_log_page_iowrites         =          0
		Num_log_append_records        =          0
		Num_log_archives              =          0
		Num_log_checkpoints           =          0
		Num_log_wals                  =          0
		Num_page_locks_acquired       =          0
		Num_object_locks_acquired     =          0
		Num_page_locks_converted      =          0
		Num_object_locks_converted    =          0
		Num_page_locks_re-requested   =          0
		Num_object_locks_re-requested =          0
		Num_page_locks_waits          =          0
		Num_object_locks_waits        =          0
		Num_tran_commits              =          0
		Num_tran_rollbacks            =          0
		Num_tran_savepoints           =          0
		Num_tran_start_topops         =          0
		Num_tran_end_topops           =          0
		Num_tran_interrupts           =          0
		Num_btree_inserts             =          0
		Num_btree_deletes             =          0
		Num_btree_updates             =          0
		Num_btree_covered             =          0
		Num_btree_noncovered          =          0
		Num_btree_resumes             =          0
		Num_btree_multirange_optimization =      0
		Num_query_selects             =          0
		Num_query_inserts             =          0
		Num_query_deletes             =          0
		Num_query_updates             =          0
		Num_query_sscans              =          0
		Num_query_iscans              =          0
		Num_query_lscans              =          0
		Num_query_setscans            =          0
		Num_query_methscans           =          0
		Num_query_nljoins             =          0
		Num_query_mjoins              =          0
		Num_query_objfetches          =          0
		Num_network_requests          =          1
		Num_adaptive_flush_pages      =          0
		Num_adaptive_flush_log_pages  =          0
		Num_adaptive_flush_max_pages  =        900
		 
		 *** OTHER STATISTICS ***
		Data_page_buffer_hit_ratio    =       0.00


	The followings are the explanation about the above statistical informations

	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Category         | Item                                   | Description                                                                          |
	+==================+========================================+======================================================================================+
	| File I/O         | Num_file_removes                       | The number of files removed                                                          |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_file_creates                       | The number of files created                                                          |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_file_ioreads                       | The number of files read                                                             |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_file_iowrites                      | The number of files stored                                                           |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_file_iosynches                     | The number of file synchronization                                                   |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Page buffer      | Num_data_page_fetches                  | The number of pages fetched                                                          |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_data_page_dirties                  | The number of duty pages                                                             |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_data_page_ioreads                  | The number of pages read                                                             |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_data_page_iowrites                 | The number of pages stored                                                           |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_data_page_victims                  | The number specifying the victim data to be flushed from the data page to the disk   |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_data_page_iowrites_for_replacement | The number of the written data pages specified as victim                             |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_adaptive_flush_pages               | The number of data pages flushed from the data buffer to the disk                    |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_adaptive_flush_log_pages           | The number of log pages flushed from the log buffer to the disk                      |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_adaptive_flush_max_pages           | The maximum number of pages allowed to flush from data and the log buffer            |
	|                  |                                        | to the disk                                                                          |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Logs             | Num_log_page_ioreads                   | The number of log pages read                                                         |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_log_page_iowrites                  | The number of log pages stored                                                       |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_log_append_records                 | The number of log records appended                                                   |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_log_archives                       | The number of logs archived                                                          |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_log_checkpoints                    | The number of checkpoints                                                            |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_log_wals                           | Not used                                                                             |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Transactions     | Num_tran_commits                       | The number of commits                                                                |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_tran_rollbacks                     | The number of rollbacks                                                              |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_tran_savepoints                    | The number of savepoints                                                             |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_tran_start_topops                  | The number of top operations started                                                 |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_tran_end_topops                    | The number of top perations stopped                                                  |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_tran_interrupts                    | The number of interruptions                                                          |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Concurrency/lock | Num_page_locks_acquired                | The number of locked pages acquired                                                  |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_object_locks_acquired              | The number of locked objects acquired                                                |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_page_locks_converted               | The number of locked pages converted                                                 |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_object_locks_converted             | The number of locked objects converted                                               |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_page_locks_re-requested            | The number of locked pages requested                                                 |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_object_locks_re-requested          | The number of locked objects requested                                               |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_page_locks_waits                   | The number of locked pages waited                                                    |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_object_locks_waits                 | The number of locked objects waited                                                  |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Index            | Num_btree_inserts                      | The number of nodes inserted                                                         |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_btree_deletes                      | The number of nodes deleted                                                          |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_btree_updates                      | The number of nodes updated                                                          |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_btree_covered                      | The number of cases in which an index includes all data upon query execution         |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_btree_noncovered                   | The number of cases in which an index includes some or no data upon query execution  |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_btree_resumes                      | The exceeding number of index scan specified in index_scan_oid_buffer_pages          |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_btree_multirange_optimization      | The number of executions on multi-range optimization for the WHERE … IN …            |
	|                  |                                        | LIMIT condition query statement                                                      |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Query            | Num_query_selects                      | The number of SELECT query execution                                                 |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_inserts                      | The number of INSERT query execution                                                 |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_deletes                      | The number of DELETE query execution                                                 |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_updates                      | The number of UPDATE query execution                                                 |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_sscans                       | The number of sequential scans (full scan)                                           |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_iscans                       | The number of index scans                                                            |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_lscans                       | The number of LIST scans                                                             |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_setscans                     | The number of SET scans                                                              |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_methscans                    | The number of METHOD scans                                                           |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_nljoins                      | The number of nested loop joins                                                      |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_mjoins                       | The number of parallel joins                                                         |
	|                  +----------------------------------------+--------------------------------------------------------------------------------------+
	|                  | Num_query_objfetches                   | The number of fetch objects                                                          |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Network request  | Num_network_requests                   | The number of network requested                                                      |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+
	| Buffer hit rate  | Data_page_buffer_hit_ratio             | Hit Ratio of page buffers                                                            |
	|                  |                                        | (Num_data_page_fetches - Num_data_page_ioreads)*100 / Num_data_page_fetches          |
	+------------------+----------------------------------------+--------------------------------------------------------------------------------------+

.. option:: -o, --output-file=FILE


	**-o** options is used to store statistics information of server processing for the database to a specified file.  ::

		cubrid statdump -o statdump.log testdb

.. option:: -c, --cumulative

	You can display the accumulated operation statistics information of the target database server by using the **-c** option. By combining this with the -i option, you can check the operation statistics information at a specified interval.  ::

		cubrid statdump -i 5 -c testdb

.. option::  -s, --substr

	You can display statistics about items of which name include the specified string by using **-s** option. The following example shows how to display statistics about items of which name include "data".
 
	::
	
		cubrid statdump -s data testdb

		*** SERVER EXECUTION STATISTICS ***
		Num_data_page_fetches         =        135
		Num_data_page_dirties         =          0
		Num_data_page_ioreads         =          0
		Num_data_page_iowrites        =          0
		Num_data_page_victims         =          0
		Num_data_page_iowrites_for_replacement =          0
		 
		 *** OTHER STATISTICS ***
		Data_page_buffer_hit_ratio    =     100.00

 

.. note::

	Each status information consists of 64-bit INTEGER data and the corresponding statistics information can be lost if the accumulated value exceeds the limit.

Checking Lock Status
====================

The **cubrid lockdb** utility is used to check the information on the lock being used by the current transaction in the database. ::

	cubrid lockdb [options] database_name

*  **cubrid**: An integrated utility for the CUBRID service and database management.

*  **lockdb**: A command used to check the information on the lock being used by the current transaction in the database.

*  *database_name*: The name of the database where lock information of the current transaction is to be checked.

The following example shows how to display lock information of the *testdb* database on a screen without any option. ::

	cubrid lockdb testdb

The following shows [options] available with the **cubrid statdump** utility.
	
.. program:: lockdb

.. option:: -o
	
	The **-o** option displays the lock information of the *testdb* database as a output.txt. ::

		cubrid lockdb -o output.txt testdb

		
Output Contents
---------------

The output contents of **cubrid lockdb** are divided into three logical sections.

*  Server lock settings

*  Clients that are accessing the database

*  The contents of an object lock table

**Server lock settings**

The first section of the output of **cubrid lockdb** is the database lock settings.

::

	*** Lock Table Dump ***
	 Lock Escalation at = 100000, Run Deadlock interval = 0

The lock escalation level is 100,000 records, and the interval to detect deadlock is set to 0 seconds (For a description of the related system parameters, **lock_escalation** and **deadlock_detection_interval**, see `Concurrency/Lock-Related Parameters <#pm_pm_db_classify_lock_htm>`_ ).

**Clients that are accessing the database**

The second section of the output of **cubrid lockdb** includes information on all clients that are connected to the database. This includes the transaction index, program name, user ID, host name, process ID, isolation level and lock timeout settings of each client.

::

	Transaction (index 1, csql, dba@cubriddb|12854)
	Isolation READ COMMITTED CLASSES AND READ UNCOMMITTED INSTANCES
	Timeout_period -1

Here, the transaction index is 1, the program name is csql, the user ID is dba, the host name is cubriddb, the client process identifier is 12854, the isolation level is READ COMMITTED CLASSES AND READ UNCOMMITTED INSTANCES, and the lock timeout is unlimited.

A client for which transaction index is 0 is the internal system transaction. It can obtain the lock at a specific time, such as the processing of a checkpoint by a database. In most cases, however, this transaction will not obtain any locks.

Because **cubrid lockdb** utility accesses the database to obtain the lock information, the **cubrid lockdb** is an independent client and will be output as such.

**Object lock table**

The third section of the output of the **cubrid lockdb** includes the contents of the object lock table. It shows which client has the lock for which object in which mode, and which client is waiting for which object in which mode. The first part of the result of the object lock table shows how many objects are locked.

::

	Object lock Table:
	    Current number of ojbects which are locked = 2001

**cubrid lockdb** outputs the OID, object type and table name of each object that obtained lock. In addition, it outputs the number of transactions that hold lock for the object (Num holders), the number of transactions (Num blocked-holders) that hold lock but are blocked since it could not convert the lock to the upper lock (e.g., conversion from U_LOCK to X_LOCK), and the number of different transactions that are waiting for the lock of the object (Num waiters). It also outputs the list of client transactions that hold lock, blocked client transactions and waiting client transactions.

The example below shows an object in which the object type is an instance of a class, or record that will be blocked, because the OID( 2| 50| 1) object that has S_LOCK for transaction 1 and S_LOCK for transaction 2 cannot be converted into X_LOCK. It also shows that transaction 3 is blocked because transaction 2 is waiting for X_LOCK even when transaction 3 is wating for S_LOCK.

::

	OID = 2| 50| 1
	Object type: instance of class ( 0| 62| 5) = athlete
	Num holders = 1, Num blocked-holders= 1, Num waiters = 1
	LOCK HOLDERS :
		Tran_index = 2, Granted_mode = S_LOCK, Count = 1
	BLOCKED LOCK HOLDERS :
		Tran_index = 1, Granted_mode = U_LOCK, Count = 3
		Blocked_mode = X_LOCK
						Start_waiting_at = Fri May 3 14:44:31 2002
						Wait_for _nsecs = -1
	LOCK WAITERS :
		Tran_index = 3, Blocked_mode = S_LOCK
						Start_waiting_at = Fri May 3 14:45:14 2002
						Wait_for_nsecs = -1

It outputs the lock information on the index of the table when the object type is the Index key of class (index key).

::

	OID = -662|   572|-32512
	Object type: Index key of class ( 0|   319|  10) = athlete.
	Index name: pk_athlete_code
	Total mode of holders =   NX_LOCK, Total mode of waiters = NULL_LOCK.
	Num holders=  1, Num blocked-holders=  0, Num waiters=  0
	LOCK HOLDERS:
		Tran_index =   1, Granted_mode =  NX_LOCK, Count =   1

Granted_mode refers to the mode of the obtained lock, and Blocked_mode refers to the mode of the blocked lock. Starting_waiting_at refers to the time at which the lock was requested, and Wait_for_nsecs refers to the waiting time of the lock. The value of Wait_for_nsecs is determined by lock_timeout_in_secs, a system parameter.

When the object type is a class (table), Nsubgranules is displayed, which is the sum of the record locks and the key locks obtained by a specific transaction in the table.

::

	OID = 0| 62| 5
	Object type: Class = athlete
	Num holders = 2, Num blocked-holders= 0, Num waiters= 0
	LOCK HOLDERS:
	Tran_index = 3, Granted_mode = IS_LOCK, Count = 2, Nsubgranules = 0
	Tran_index = 1, Granted_mode = IX_LOCK, Count = 3, Nsubgranules = 1
	Tran_index = 2, Granted_mode = IS_LOCK, Count = 2, Nsubgranules = 1
	
Checking Database Consistency
=============================

The **cubrid checkdb** utility is used to check the consistency of a database. You can use **cubrid checkdb** to identify data structures that are different from indexes by checking the internal physical consistency of the data and log volumes. If the **cubrid checkdb** utility reveals any inconsistencies, you must try automatic repair by using the --**repair** option.

cubrid checkdb [options] database_name [class_name1 class_name2 ...]

	* **cubrid**: An integrated utility for CUBRID service and database management.

	* **checkdb**: A utility that checks the data consistency of a specific database.

	* *database_name*: The name of the database whose consistency status will be either checked or restored.

	*table_list.txt*: A file name to store the list of the tables for consistency check or recovery

	*class_name1 class_name2*: List the table names for consistency check or recovery

	
The following shows [options] available with the **cubrid checkdb** utility.

.. program:: checkdb

.. option::	-S, --SA-mode

	The **-S** option is used to access a database in standalone, which means it works without processing server; it does not have an argument. If **-S** is not specified, the system recognizes that a database is running in client/server mode. ::

		cubrid checkdb -S testdb

.. option:: -C, --CS-mode

	The **-C** option is used to access a database in client/server mode, which means it works in client/server process respectively; it does not have an argument. If
	**-C** is not specified, the system recognize that a database is running in client/server mode by default. ::

		cubrid checkdb -C testdb

.. option:: -r, --repair

	The **-r** option is used to restore an issue if a consistency error occurs in a database. ::

		cubrid checkdb -r testdb

.. option:: -i, --input-class-file or table name

	You can specify a table in which consistency is check or restored by specifying the **-i** *table_list.txt* option or listing the table names after a database name. In this way, you can limit the target to be restored and both ways can be used. If a specific target is not specified, entire database will be a target of consistency check or restoration. ::

		cubrid checkdb testdb tbl1 tbl2
		cubrid checkdb -r testdb tbl1 tbl2
		cubrid checkdb -r -i table_list.txt testdb tbl1 tbl2

	Empty string, tab, carriage return and comma are separators among table names in the table list file specified by **-i** option. The following example shows the table list file; from t1 to t10, it is recognized as a table for consistency check or restoration. ::

		t1 t2 t3,t4 t5
		t6, t7 t8   t9
		 
			 t10

Killing Database Transactions
=============================

The **cubrid killtran** is used to check transactions or abort specific transaction. Only a DBA can execute this utility. ::

	cubrid killtran [options] database_name

* **cubrid**: An integrated utility for the CUBRID service and database management

* **killtran**: A utility that manages transactions for a specified database

* *database_name*: The name of database whose transactions are to be killed

Some options refer to killing specified transactions; others refer to outputting active transactions. If no option is specified, **-d** is specified by default so all transactions are displayed on the screen.
 
::

	cubrid killtran testdb 
	 
	Tran index      User name   Host name      Process id      Program name
	-------------------------------------------------------------------------------
		  1(+)            dba      myhost             664           cub_cas
		  2(+)            dba      myhost            6700              csql
		  3(+)            dba      myhost            2188           cub_cas
		  4(+)            dba      myhost             696              csql
		  5(+)         public      myhost            6944              csql
	-------------------------------------------------------------------------------


The following shows [options] available with the **cubrid killtran** utility.

.. program:: killtran

.. option :: -i, --kill-transation-index=INDEX

	This option kills transactions in a specified index. ::

		cubrid killtran -i 1 testdb
		
		Ready to kill the following transactions:
		 
		Tran index      User name      Host name      Process id      Program name
		-------------------------------------------------------------------------------
			  1(+)            dba      myhost            4760              csql
		-------------------------------------------------------------------------------
		Do you wish to proceed ? (Y/N)y
		Killing transaction associated with transaction index 1
 
.. option:: --kill-user-name=ID

	This option kills transactions for a specified OS user ID. ::

		cubrid killtran --kill-user-name=os_user_id testdb

.. option::  --kill- host-name=HOST

	This opotion kills transactions of a specified client host. ::

		cubrid killtran --kill-host-name=myhost testdb

.. option:: --kill-program-name=NAME

	This option kills transactions for a specified program.  ::
	
		cubrid killtran --kill-program-name=cub_cas testdb

.. option:: -p PASSWORD
		
	A value followed by the -p option is a password of the **DBA**, and should be entered in the prompt.

.. option:: -d, --display

	The **-d** option is specified, all transactions are displayed on the screen. 
	
	::

		cubrid killtran -d testdb
  
		Tran index      User name      Host name      Process id      Program name
		-------------------------------------------------------------------------------
			  2(+)            dba      myhost            6700              csql
			  3(+)            dba      myhost            2188           cub_cas
			  4(+)            dba      myhost             696              csql
			  5(+)         public      myhost            6944              csql
		-------------------------------------------------------------------------------

.. option:: -f, --force

	This option omits a prompt to check transactions to be stopped. ::

		cubrid killtran -f -i 1 testdb

Checking the Query Plan Cache
=============================

The **cubrid plandump** utility is used to display information on the query plans stored (cached) on the server. ::

	cubrid plandump options database_name 

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **plandump**: A utility that displays the query plans stored in the current cache of a specific database.

* *database_name*: The name of the database where the query plans are to be checked or dropped from its sever cache.

If no option is used, it checks the query plans stored in the cache. ::

	cubrid plandump testdb
 
The following shows [options] available with the **cubrid plandump** utility.

.. program :: plandump

.. option:: -d, --drop
 
	This option drops the query plans stored in the cache. ::

		cubrid plandump -d testdb

.. option:: -o, --output-file=FILE

	This option stores the results of the query plans stored in the cache to a file. ::

		cubrid plandump -o output.txt testdb

Outputting Internal Database Information
========================================

You can check various pieces of internal information on the database with the **cubrid diagdb** utility. Information provided by **cubrid diagdb** is helpful in diagnosing the current status of the database or figuring out a problem. ::

	cubrid diagdb options database_name

* **cubrid**: An integrated utility for the CUBRID service and database management.

* **diagdb**: A command that is used to check the current storage state of the database by outputting the information contained in the binary file managed by CUBRID in text format. It normally executes only when the database is in a stopped state. You can check the whole database or the file table, file size, heap size, class name or disk bitmap selectively by using the provided option.

* *database_name*: The name of the database of which internal information is to be diagnosed.

The following shows [options] available with the **cubrid diagdb** utility.

.. program:: diagdb

.. option:: -d, --dump-type=TYPE

	This option specifies the output range when you display the information of all files in the *testdb* database. If any option is not specified, the default value of 1 is used.

		cubrid diagdb -d 1 myhost testdb

	The utility has 9 types of -d options as follows:

	+------+--------------------------------------+
	| Type | Description                          |
	+------+--------------------------------------+
	| -1   | Displays all database information.   |
	+------+--------------------------------------+
	| 1    | Displays file table information.     |
	+------+--------------------------------------+
	| 2    | Displays file capacity information.  |
	+------+--------------------------------------+
	| 3    | Displays heap capacity information.  |
	+------+--------------------------------------+
	| 4    | Displays index capacity information. |
	+------+--------------------------------------+
	| 5    | Displays class name information.     |
	+------+--------------------------------------+
	| 6    | Displays disk bitmap information.    |
	+------+--------------------------------------+
	| 7    | Displays catalog information.        |
	+------+--------------------------------------+
	| 8    | Displays log information.            |
	+------+--------------------------------------+
	| 9    | Displays hip information.            |
	+------+--------------------------------------+

Backing up and Restoring
========================

**DBA** must perform regular backups of the database so that it can be restored successfully to a state at a certain point in time in case of system failure. For details, see `Database Backup <#admin_admin_br_backup_htm>`_ .

Exporting and Importing
=======================

To use a newer version of CUBRID database, the existing version must be migrated to a new one. For this purpose, you can use "Export to an ASCII text file" and "Import from an ASCII text file" features provided by CUBRID. For details on export and import, see `Migrating Database <#admin_admin_migration_migration__1472>`_ .

Dumping Parameters Used in Server/Client
=========================================

The **cubrid paramdump** utility outputs parameter information used in the server/client process.

	cubrid paramdump [options] database_name

* **cubrid**: An integrated utility for the CUBRID service and database management

* **paramdump**: A utility that outputs parameter information used in the server/client process

* *options*: A short name option starts with a single dash ( **-** ) while a full name option starts with a double dash ( **--** ). **-o**, **-b**, **-S** and **-C** options are provided.

* *database_name*: The name of the database in which parameter information is to be displayed.

The following shows [options] available with the **cubrid paramdump** utility.

.. program:: paramdump

.. option:: -o, --output-file=FILE

	The **-o** option is used to store information of the parameters used in the server/client process of the database into a specified file. The file is created in the current directory. If the **-o** option is not specified, the message is displayed on a console screen. ::

		cubrid paramdump -o db_output testdb

.. option:: -b, --both

	The **-b** option is used to display parameter information used in server/client process on a console screen. If the **-b** option is not specified, only server-side information is displayed. ::
 	
		cubrid paramdump -b testdb

.. option:: -S, --SA-mode

	This option displays parameter information of the server process in standalone mode. ::

		cubrid paramdump -S testdb

.. option:: -C, --CS-mode

	This option displays parameter information of the server process in client/server mode. ::

		cubrid paramdump -C testdb

Locale Compile/Output
=====================

**cubrid genlocale** utility compiles the locale information to use. This utility is executed in the **make_locale.sh** script ( **.bat** for Windows).

**cubrid dumplocale** utility outputs the compiled binary locale file as a human-readable format on the console. The output value may be very large, so we recommend that you save the value as a file by redirecting.

For more detailed usage, see `Locale Setting <#admin_admin_i18n_locale_htm>`_ .