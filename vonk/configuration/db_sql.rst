.. |br| raw:: html

   <br />
   
.. _configure_sql:

Using SQL server
================

There are two ways to create the Vonk database on a SQL Server instance: Have Vonk create it for you entirely or create an empty database and users yourself and have Vonk create the schema (tables etc.).

In both cases:

*   Prepare an instance of SQL Server 2012 or newer. Any edition - including SQL Server Express - will do.
    The instance will have a servername and possibly an instancename: ``server/instance``.

*	In a text editor open :code:`appsettings.json` to find the ``Repository`` setting::

	"Repository": "Memory",

*	Change the setting to ``SQL``

*   Find the section called ``SqlDbOptions``. It has these values by default::

        "SqlDbOptions": {
            "ConnectionString": "connectionstring to your Vonk SQL Server database (SQL2012 or newer); Set MultipleActiveResultSets=True",
            "SchemaName": "vonk",
            "AutoUpdateDatabase": true
            //"AutoUpdateConnectionString" : "set this to the same database as 'ConnectionString' but with credentials that can alter the database. If not set, defaults to the value of 'ConnectionString'"
        },

*   The site `connectionstrings.com <https://www.connectionstrings.com/sqlconnection/>`_ is useful for determining the correct connectionstring for your environment.

*   If you will only use Windows Accounts, you can use the (default) Authentication Mode, which is Windows Authentication Mode. But if you also want to use SQL Server accounts, you have to run it in Mixed Mode. Refer to `Authentication in SQL Server <https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql/authentication-in-sql-server>`_ for more information.
Have Vonk create your database
------------------------------

This option is mainly for experimentation as it effectively requires sysadmin privileges for the connecting user.

*   Prepare a login on SQL Server with the following role:

    *   sysadmin

*   Set the ``SqlDbOptions`` for the Vonk database as follows (the values are example values for connecting with your own Windows login):
    ::

        "SqlDbOptions": {
            "ConnectionString": "Integrated Security=SSPI;Persist Security Info=False;Initial Catalog=VonkStu3;Data Source=Server\Instance;MultipleActiveResultSets=true",
            "SchemaName": "vonk",
            "AutoUpdateDatabase": true
        },

*   Set the ``SqlDbOptions`` under ``Administration`` for the Administration database likewise:
    ::

        "Administration": {
            "Repository": "SQL",
            "SqlDbOptions": {
                "ConnectionString": "Integrated Security=SSPI;Persist Security Info=False;Initial Catalog=VonkStu3;Data Source=Server\Instance;MultipleActiveResultSets=true",
                "SchemaName": "vonk",
                "AutoUpdateDatabase": true
            }
        }

*   You don't need to set AutoUpdateConnectionString since the ConnectionString will already have enough permissions.

*   Start Vonk. It will display in its log that it applied pending migrations. After that the database is created and set up with the correct schema.

Create a database and users by script, and have Vonk create the schema
----------------------------------------------------------------------

*   Log into SQL Server as the Administrator user.

*	From the working directory open :code:`data\01-CreateDatabases.sql`

*	In SQL Server Management Studio, in the menu select Query|SQLCMD Mode.

*	In the script uncomment and adjust the variable names :code:`dbName` and :code:`AdminDbName` as well as any other variables to your own liking.

*   Run the script to create both the Vonk database and the Administration API database.

*	From the working directory open :code:`data\02-CreateDBUser.sql`

*	In SQL Server Management Studio, in the menu select Query|SQLCMD Mode.

*	In the script uncomment and adjust the variables at the top names to your own liking.

*   Run the script to create two users, one with access to the Vonk database, the other with access to the Administration database.
    This script grants the database role db_ddladmin to both users, to enable the AutoUpdateDatabase feature.
    Refer to `Overview of permissions`_ for an overview of neccessary authorization for different features.

*   Set the ``SqlDbOptions`` for the Vonk database as follows:
    ::

        "SqlDbOptions": {
            "ConnectionString": "User Id=<dbUserName>;Password=<dbPassword>;Initial Catalog=<dbName>;Data Source=server\\instance;MultipleActiveResultSets=True",
            "SchemaName": "vonk",
            "AutoUpdateDatabase": "true"
        }

*   If you have set up a different user for running the AutoUpdateDatabase feature, you can provide that:
    ::

        "SqlDbOptions": {
            "ConnectionString": "User Id=<dbUserName>;Password=<dbPassword>;Initial Catalog=<dbName>;Data Source=server\\instance;MultipleActiveResultSets=True",
            "SchemaName": "vonk",
            "AutoUpdateDatabase": "true"
            "AutoUpdateConnectionString": "User Id=<updateUserName>;Password=<updatePassword>;Initial Catalog=<dbName>;Data Source=server\\instance;MultipleActiveResultSets=True",
        }

*   Set the ``SqlDbOptions`` under ``Administration`` for the Administration database likewise:
    ::
	
        "Administration" : {
            "Repository": "SQL",
            "SqlDbOptions": {
                "ConnectionString": "User Id=<AdminDbUserName>;Password=<AdminDbPassword>;Initial Catalog=<AdminbName>;Data Source=server\\instance;MultipleActiveResultSets=True",
                "SchemaName": "vonk",
                "AutoUpdateDatabase": "true"
            }
        }

*   For the administration you can also provide different credentials for performing the auto update:
    ::

        "Administration" : {
            "Repository": "SQL",
            "SqlDbOptions": {
                "ConnectionString": "User Id=<AdminDUserName>;Password=<AdminDbPassword>;Initial Catalog=<AdminbName>;Data Source=server\\instance;MultipleActiveResultSets=True",
                "SchemaName": "vonk",
                "AutoUpdateDatabase": "true"
                "AutoUpdateConnectionString": "User Id=<updateAdminUserName>;Password=<updateAdminPassword>;Initial Catalog=<AdminDbName>;Data Source=server\\instance;MultipleActiveResultSets=True",
            }
        }

Overview of permissions
-----------------------
This paragraph lists the permissions needed to perform specific actions on the SQL database.

*   To run the AutoUpdateDatabase feature, including creation of the databases:

    *   **sysadmin**

*   To run the AutoUpdateDatabase feature on an already created (but empty) database:

    *   **db_ddladmin** |br|
        (both for the normal Vonk database and the Administration database)

*   To read/write resources:

    *   **db_datareader**
    *   **db_datawriter**

*   To execute the ResetDb feature:

    *   **db_ddladmin** |br|
        only on the normal Vonk database for the user in the SqlDbOptions connectionstring. |br|
        (no extra permissions are required for the user on the Administration database).


