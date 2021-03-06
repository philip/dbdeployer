# dbdeployer

[DBdeployer](https://github.com/datacharmer/dbdeployer) is a tool that deploys MySQL database servers easily.
This is a port of [MySQL-Sandbox](https://github.com/datacharmer/mysql-sandbox), originally written in Perl, and re-designed from the ground up in [Go](https://golang.org). See the [features comparison](https://github.com/datacharmer/dbdeployer/blob/master/docs/features.md) for more detail.

Documentation updated for version 1.6.0 (19-Jun-2018 21:14 UTC)

## Installation

The installation is simple, as the only thing you will need is a binary executable for your operating system.
Get the one for your O.S. from [dbdeployer releases](https://github.com/datacharmer/dbdeployer/releases) and place it in a directory in your $PATH.
(There are no binaries for Windows. See the [features list](https://github.com/datacharmer/dbdeployer/blob/master/docs/features.md) for more info.)

For example:

    $ VERSION=1.6.0
    $ origin=https://github.com/datacharmer/dbdeployer/releases/download/$VERSION
    $ wget $origin/dbdeployer-$VERSION.linux.tar.gz
    $ tar -xzf dbdeployer-$VERSION.linux.tar.gz
    $ chmod +x dbdeployer-$VERSION.linux
    $ sudo mv dbdeployer-$VERSION.linux /usr/local/bin/dbdeployer

Of course, there are **prerequisites**: your machine must be able to run the MySQL server. Be aware that version 5.6+ and higher require some libraries that are not installed by default in all flavors of Linux (libnuma, libaio.)

## Main operations

(See this ASCIIcast for a demo of its operations.)
[![asciicast](https://asciinema.org/a/165707.png)](https://asciinema.org/a/165707)

With dbdeployer, you can deploy a single sandbox, or many sandboxes  at once, with or without replication.

The main command is ``deploy`` with its subcommands ``single``, ``replication``, and ``multiple``, which work with MySQL tarball that have been unpacked into the _sandbox-binary_ directory (by default, $HOME/opt/mysql.)

To use a tarball, you must first run the ``unpack`` command, which will unpack the tarball into the right directory.

For example:

    $ dbdeployer unpack mysql-8.0.4-rc-linux-glibc2.12-x86_64.tar.gz
    Unpacking tarball mysql-8.0.4-rc-linux-glibc2.12-x86_64.tar.gz to $HOME/opt/mysql/8.0.4
    .........100.........200.........292

    $ dbdeployer deploy single 8.0.4
    Database installed in $HOME/sandboxes/msb_8_0_4
    . sandbox server started


The program doesn't have any dependencies. Everything is included in the binary. Calling *dbdeployer* without arguments or with ``--help`` will show the main help screen.

    $ dbdeployer --version
    dbdeployer version 1.6.0
    

    $ dbdeployer -h
    dbdeployer makes MySQL server installation an easy task.
    Runs single, multiple, and replicated sandboxes.
    
    Usage:
      dbdeployer [command]
    
    Available Commands:
      admin       sandbox management tasks
      defaults    tasks related to dbdeployer defaults
      delete      delete an installed sandbox
      deploy      deploy sandboxes
      global      Runs a given command in every sandbox
      help        Help about any command
      sandboxes   List installed sandboxes
      unpack      unpack a tarball into the binary directory
      usage       Shows usage of installed sandboxes
      versions    List available versions
    
    Flags:
          --config string           configuration file (default "$HOME/.dbdeployer/config.json")
      -h, --help                    help for dbdeployer
          --sandbox-binary string   Binary repository (default "$HOME/opt/mysql")
          --sandbox-home string     Sandbox deployment directory (default "$HOME/sandboxes")
          --version                 version for dbdeployer
    
    Use "dbdeployer [command] --help" for more information about a command.
    

The flags listed in the main screen can be used with any commands.
The flags ``--my-cnf-options`` and ``--init-options`` can be used several times.

If you don't have any tarballs installed in your system, you should first ``unpack`` it (see an example above).

    $ dbdeployer unpack -h
    If you want to create a sandbox from a tarball, you first need to unpack it
    into the sandbox-binary directory. This command carries out that task, so that afterwards 
    you can call 'deploy single', 'deploy multiple', and 'deploy replication' commands with only 
    the MySQL version for that tarball.
    If the version is not contained in the tarball name, it should be supplied using --unpack-version.
    If there is already an expanded tarball with the same version, a new one can be differentiated with --prefix.
    
    Usage:
      dbdeployer unpack MySQL-tarball [flags]
    
    Aliases:
      unpack, extract, untar, unzip, inflate, expand
    
    Examples:
    
        $ dbdeployer unpack mysql-8.0.4-rc-linux-glibc2.12-x86_64.tar.gz
        Unpacking tarball mysql-8.0.4-rc-linux-glibc2.12-x86_64.tar.gz to $HOME/opt/mysql/8.0.4
    
        $ dbdeployer unpack --prefix=ps Percona-Server-5.7.21-linux.tar.gz
        Unpacking tarball Percona-Server-5.7.21-linux.tar.gz to $HOME/opt/mysql/ps5.7.21
    
        $ dbdeployer unpack --unpack-version=8.0.18 --prefix=bld mysql-mybuild.tar.gz
        Unpacking tarball mysql-mybuild.tar.gz to $HOME/opt/mysql/bld8.0.18
    	
    
    Flags:
      -h, --help                    help for unpack
          --prefix string           Prefix for the final expanded directory
          --unpack-version string   which version is contained in the tarball
          --verbosity int           Level of verbosity during unpack (0=none, 2=maximum) (default 1)
    
    

The easiest command is ``deploy single``, which installs a single sandbox.

    $ dbdeployer deploy -h
    Deploys single, multiple, or replicated sandboxes
    
    Usage:
      dbdeployer deploy [command]
    
    Available Commands:
      multiple    create multiple sandbox
      replication create replication sandbox
      single      deploys a single sandbox
    
    Flags:
          --base-port int                 Overrides default base-port (for multiple sandboxes)
          --binary-version string         Specifies the version when the basedir directory name does not contain it (i.e. it is not x.x.xx)
          --bind-address string           defines the database bind-address  (default "127.0.0.1")
          --concurrent                    Runs multiple sandbox deployments concurrently
          --custom-mysqld string          Uses an alternative mysqld (must be in the same directory as regular mysqld)
      -p, --db-password string            database password (default "msandbox")
      -u, --db-user string                database user (default "msandbox")
          --defaults strings              Change defaults on-the-fly (--defaults=label:value)
          --disable-mysqlx                Disable MySQLX plugin (8.0.11+)
          --enable-general-log            Enables general log for the sandbox (MySQL 5.1+)
          --enable-mysqlx                 Enables MySQLX plugin (5.7.12+)
          --expose-dd-tables              In MySQL 8.0+ shows data dictionary tables
          --force                         If a destination sandbox already exists, it will be overwritten
          --gtid                          enables GTID
      -h, --help                          help for deploy
          --init-general-log              uses general log during initialization (MySQL 5.1+)
      -i, --init-options strings          mysqld options to run during initialization
          --keep-server-uuid              Does not change the server UUID
          --my-cnf-file string            Alternative source file for my.sandbox.cnf
      -c, --my-cnf-options strings        mysqld options to add to my.sandbox.cnf
          --native-auth-plugin            in 8.0.4+, uses the native password auth plugin
          --port int                      Overrides default port
          --post-grants-sql strings       SQL queries to run after loading grants
          --post-grants-sql-file string   SQL file to run after loading grants
          --pre-grants-sql strings        SQL queries to run before loading grants
          --pre-grants-sql-file string    SQL file to run before loading grants
          --remote-access string          defines the database access  (default "127.%")
          --rpl-password string           replication password (default "rsandbox")
          --rpl-user string               replication user (default "rsandbox")
          --sandbox-directory string      Changes the default sandbox directory
          --skip-load-grants              Does not load the grants
          --skip-report-host              Does not include report host in my.sandbox.cnf
          --skip-report-port              Does not include report port in my.sandbox.cnf
          --skip-start                    Does not start the database server
          --use-template strings          [template_name:file_name] Replace existing template with one from file
    
    

    $ dbdeployer deploy single -h
    single installs a sandbox and creates useful scripts for its use.
    MySQL-Version is in the format x.x.xx, and it refers to a directory named after the version
    containing an unpacked tarball. The place where these directories are found is defined by 
    --sandbox-binary (default: $HOME/opt/mysql.)
    For example:
    	dbdeployer deploy single 5.7     # deploys the latest release of 5.7.x
    	dbdeployer deploy single 5.7.21  # deploys a specific release
    
    For this command to work, there must be a directory $HOME/opt/mysql/5.7.21, containing
    the binary files from mysql-5.7.21-$YOUR_OS-x86_64.tar.gz
    Use the "unpack" command to get the tarball into the right directory.
    
    Usage:
      dbdeployer deploy single MySQL-Version [flags]
    
    Flags:
      -h, --help     help for single
          --master   Make the server replication ready
    
    

If you want more than one sandbox of the same version, without any replication relationship, use the ``deploy multiple`` command with an optional ``--nodes`` flag (default: 3).

    $ dbdeployer deploy multiple -h
    Creates several sandboxes of the same version,
    without any replication relationship.
    For this command to work, there must be a directory $HOME/opt/mysql/5.7.21, containing
    the binary files from mysql-5.7.21-$YOUR_OS-x86_64.tar.gz
    Use the "unpack" command to get the tarball into the right directory.
    
    Usage:
      dbdeployer deploy multiple MySQL-Version [flags]
    
    Examples:
    
    	$ dbdeployer deploy multiple 5.7.21
    	
    
    Flags:
      -h, --help        help for multiple
      -n, --nodes int   How many nodes will be installed (default 3)
    
    

The ``deploy replication`` command will install a master and two or more slaves, with replication started. You can change the topology to *group* and get three nodes in peer replication, or compose multi-source topologies with *all-masters* or *fan-in*.

    $ dbdeployer deploy replication -h
    The replication command allows you to deploy several nodes in replication.
    Allowed topologies are "master-slave" for all versions, and  "group", "all-masters", "fan-in"
    for  5.7.17+.
    For this command to work, there must be a directory $HOME/opt/mysql/5.7.21, containing
    the binary files from mysql-5.7.21-$YOUR_OS-x86_64.tar.gz
    Use the "unpack" command to get the tarball into the right directory.
    
    Usage:
      dbdeployer deploy replication MySQL-Version [flags]
    
    Examples:
    
    		$ dbdeployer deploy replication 5.7    # deploys highest revision for 5.7
    		$ dbdeployer deploy replication 5.7.21 # deploys a specific revision
    		# (implies topology = master-slave)
    
    		$ dbdeployer deploy --topology=master-slave replication 5.7
    		# (explicitly setting topology)
    
    		$ dbdeployer deploy --topology=group replication 5.7
    		$ dbdeployer deploy --topology=group replication 8.0 --single-primary
    		$ dbdeployer deploy --topology=all-masters replication 5.7
    		$ dbdeployer deploy --topology=fan-in replication 5.7
    	
    
    Flags:
      -h, --help                 help for replication
          --master-ip string     Which IP the slaves will connect to (default "127.0.0.1")
          --master-list string   Which nodes are masters in a multi-source deployment (default "1,2")
      -n, --nodes int            How many nodes will be installed (default 3)
          --semi-sync            Use semi-synchronous plugin
          --single-primary       Using single primary for group replication
          --slave-list string    Which nodes are slaves in a multi-source deployment (default "3")
      -t, --topology string      Which topology will be installed (default "master-slave")
    
    

## Standard and non-standard basedir names

dbdeployer expects to get the binaries from ``$HOME/opt/mysql/x.x.xx``. For example, when you run the command ``dbdeployer deploy single 8.0.11``, you must have the binaries for MySQL 8.0.11 expanded into a directory named ``$HOME/opt/mysql/8.0.11``.

If you want to keep several directories with the same version, you can differentiate them using a **prefix**:

    $HOME/opt/mysql/
                8.0.11
                lab_8.0.11
                ps_8.0.11
                myown_8.0.11

In the above cases, running ``dbdeployer deploy single lab_8.0.11`` will do what you expect, i.e. dbdeployer will use the binaries in ``lab_8.0.11`` and recognize ``8.0.11`` as the version for the database.

When the extracted tarball directory name that you want to use doesn't contain the full version number (such as ``/home/dbuser/build/path/5.7-extra``) you need to provide the version using the option ``--binary-version``. For example:

    dbdeployer deploy single 5.7-extra \
        --sandbox-binary=/home/dbuser/build/path \
        --binary-version=5.7.22

In the above command, ``--sandbox-binary`` indicates where to search for the binaries, ``5.7-extra`` is where the binaries are, and ``--binary-version`` indicates which version should be used.

## Using short version numbers

You can use, instead of a full version number (e.g. ``8.0.11``,) a short one, such as ``8.0``. This shortcut works starting with version 1.6.0.
When you invoke dbdeployer with a short number, it will look for the highest revision number within that version, and use it for deployment.

For example, if your sandbox binary directory contains the following:

    5.7.19    5.7.20    5.7.22    8.0.1    8.0.11    8.0.4

You can issue the command ``dbdeployer deploy single 8.0``, and it will use 8.0.11 for a single deployment. Or ``dbdeployer deploy replication 5.7`` and it will result in a replication system using 5.7.22 (the latest one.)


## Multiple sandboxes, same version and type

If you want to deploy several instances of the same version and the same type (for example two single sandboxes of 8.0.4, or two group replication instances with different single-primary setting) you can specify the data directory name and the ports manually.

    $ dbdeployer deploy single 8.0.4
    # will deploy in msb_8_0_4 using port 8004

    $ dbdeployer deploy single 8.0.4 --sandbox-directory=msb2_8_0_4
    # will deploy in msb2_8_0_4 using port 8005 (which dbdeployer detects and uses)

    $ dbdeployer deploy replication 8.0.4 --sandbox-directory=rsandbox2_8_0_4 --base-port=18600
    # will deploy replication in rsandbox2_8_0_4 using ports 18601, 18602, 18603

## Ports management

dbdeployer will try using the default port for each sandbox whenever possible. For single sandboxes, the port will be the version number without dots: 5.7.22 will deploy on port 5722. For multiple sandboxes, the port number is defined by using a prefix number (visible in the defaults: ``dbdeployer defaults list``) + the port number + the revision number (for some topologies multiplied by 100.)
For example, single-primary group replication with MySQL 8.0.11 will compute the ports like this:

    base port = 8011 (version number) + 13000 (prefix) + 11 (revision) * 100  = 22111
    node1 port = base port + 1 = 22112
    node2 port = base port + 2 = 22113
    node3 port = base port + 2 = 22114

For group replication we need to calculate the group port, and we use the ``group-port-delta`` (= 125) to obtain it from the regular port:

    node1 group port = 22112 + 125 = 22237
    node2 group port = 22113 + 125 = 22238
    node3 group port = 22114 + 125 = 22239

For MySQL 8.0.11+, we also need to assign a port for the XPlugin, and we compute that using the regular port + the ``mysqlx-port-delta`` (=10000).

Thus, for MySQL 8.0.11 group replication deployments, you would see this listing:

    $ dbdeployer sandboxes --header
    name                   type                  version  ports
    ----------------       -------               -------  -----
    group_msb_8_0_11     : group-multi-primary    8.0.11 [20023 20148 30023 20024 20149 30024 20025 20150 30025]
    group_sp_msb_8_0_11  : group-single-primary   8.0.11 [22112 22237 32112 22113 22238 32113 22114 22239 32114]

This method makes port clashes unlikely when using the same version in different deployments, but there is a risk of port clashes when deploying many multiple sandboxes of close-by versions.
Furthermore, dbdeployer doesn't let the clash happen. Thanks to its central catalog of sandboxes, it knows which ports were already used, and will search for free ones whenever a potential clash is detected.
Bear in mind that the concept of "used" is only related to sandboxes. dbdeployer does not know if ports may be used by other applications.
You can minimize risks by telling dbdeployer which ports may be occupied. The defaults have a field ``reserved-ports``, containing the ports that should not be used. You can add to that list by modifying the defaults. For example, if you want to exclude port 7001, 10000, and 15000 from being used, you can run

    dbdeployer defaults update reserved-ports '7001,10000,15000'

or, if you want to preserve the ones that are reserved by default:

    dbdeployer defaults update reserved-ports '1186,3306,33060,7001,10000,15000'

## Concurrent deployment and deletion

Starting with version 0.3.0, dbdeployer can deploy groups of sandboxes (``deploy replication``, ``deploy multiple``) with the flag ``--concurrent``. When this flag is used, dbdeployed will run operations concurrently.
The same flag can be used with the ``delete`` command. It is useful when there are several sandboxes to be deleted at once.
Concurrent operations run from 2 to 5 times faster than sequential ones, depending on the version of the server and the number of nodes.

## Replication topologies

Multiple sandboxes can be deployed using replication with several topologies (using ``dbdeployer deploy replication --topology=xxxxx``:

* **master-slave** is the default topology. It will install one master and two slaves. More slaves can be added with the option ``--nodes``.
* **group** will deploy three peer nodes in group replication. If you want to use a single primary deployment, add the option ``--single-primary``. Available for MySQL 5.7 and later.
* **fan-in** is the opposite of master-slave. Here we have one slave and several masters. This topology requires MySQL 5.7 or higher.
**all-masters** is a special case of fan-in, where all nodes are masters and are also slaves of all nodes.

It is possible to tune the flow of data in multi-source topologies. The default for fan-in is three nodes, where 1 and 2 are masters, and 2 are slaves. You can change the predefined settings by providing the list of components:

    $ dbdeployer deploy replication --topology=fan-in \
        --nodes=5 --master-list="1,2 3" \
        --slave-list="4,5" 8.0.4 \
        --concurrent
In the above example, we get 5 nodes instead of 3. The first three are master (``--master-list="1,2,3"``) and the last two are slaves (``--slave-list="4,5"``) which will receive data from all the masters. There is a test automatically generated to check the replication flow. In our case it shows the following:

    $ ~/sandboxes/fan_in_msb_8_0_4/test_replication
    # master 1
    # master 2
    # master 3
    # slave 4
    ok - '3' == '3' - Slaves received tables from all masters
    # slave 5
    ok - '3' == '3' - Slaves received tables from all masters
    # pass: 2
    # fail: 0

The first three lines show that each master has done something. In our case, each master has created a different table. Slaves in nodes 5 and 6 then count how many tables they found, and if they got the tables from all masters, the test succeeds.

## Skip server start

By default, when sandboxes are deployed, the servers start and additional operations to complete the topology are executed automatically. It is possible to skip the server start, using the ``--skip-start`` option. When this option is used, the server is initialized, but not started. Consequently, the default users are not created, and the database, when started manually, is only accessible with user ``root`` without password.

If you deploy with ``--skip-start``, you can run the rest of the operations manually:

    $ dbdeployer deploy single --skip-start 5.7.21
    $ $HOME/sandboxes/msb_5_7_21/start
    $ $HOME/sandboxes/msb_5_7_21/load_grants

The same can be done for replication, but you need also to run the additional step of initializing the slaves:

    $ dbdeployer deploy replication --skip-start 5.7.21 --concurrent
    $ $HOME/sandboxes/rsandbox_5_7_21/start_all
    $ $HOME/sandboxes/rsandbox_5_7_21/master/load_grants
    # NOTE: only the master needs to load grants. The slaves receive the grants through replication
    $ $HOME/sandboxes/rsandbox_5_7_21/initialize_slaves

Similarly, for group replication

    $ dbdeployer deploy replication --skip-start 5.7.21 --topology=group --concurrent
    $ $HOME/sandboxes/group_msb_5_7_21/start_all
    $ $HOME/sandboxes/group_msb_5_7_21/node1/load_grants
    $ $HOME/sandboxes/group_msb_5_7_21/node2/load_grants
    $ $HOME/sandboxes/group_msb_5_7_21/node3/load_grants
    $ $HOME/sandboxes/rsandbox_5_7_21/initialize_nodes

WARNING: running sandboxes with ``--skip-start`` is provided for advanced users and is not recommended.
If the purpose of skipping the start is to inspect the server before the sandbox granting operations, you may consider using ``--pre-grants-sql`` and ``--pre-grants-sql-file`` to run the necessary SQL commands (see _Sandbox customization_ below.)

## MySQL Document store, mysqlsh, and defaults.

MySQL 5.7.12+ introduces the XPlugin (a.k.a. _mysqlx_) which enables operations using a separate port (33060 by default) on special tables that can be treated as NoSQL collections.
In MySQL 8.0.11+ the XPlugin is enabled by default, giving dbdeployer the task of defining an additional port and socket for this service. When you deploy MySQL 8.0.11 or later, dbdeployer sets the ``mysqlx-port`` to the value of the regular port + ``mysqlx-delta-port`` (= 10000).

If you want to avoid having the XPlugin enabled, you can deploy the sandbox with the option ``--disable-mysqlx``.

For MySQL between 5.7.12 and 8.0.4, the approach is the opposite. By default, the XPlugin is disabled, and if you want to use it you will run the deployment using ``--enable-mysqlx``. In both cases the port and socket will be computed by dbdeployer.

When the XPlugin is enabled, it makes sense to use [the MySQL shell](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell.html) and dbdeployer will create a ``mysqlsh`` script for the sandboxes that use the plugin. Unfortunately, as of today (late April 2018) the MySQL shell is not released with the server tarball, and therefore we have to fix things manually. dbdeployer will look for ``mysqlsh`` in the same directory where the other clients are, so if you manually merge the mysql shell and the server tarballs, you will get the appropriate version of MySQL shell. If not, you will use the version of the shell that is available in ``$PATH``. If there is no MySQL shell available, you will get an error.

## Logs management.

Sometimes, when using sandboxes for testing, it makes sense to enable the general log, either during initialization or for regular operation. While you can do that with ``--my-cnf-options=general-log=1`` or ``--my-init-options=--general-log=1``, as of version 1.4.0 you have two simple boolean shortcuts: ``--init-general-log`` and ``--enable-general-log`` that will start the general log when requested.

Additionally, each sandbox has a convenience script named ``show_log`` that can easily display either the error log or the general log. Run `./show_log -h` for usage info.

For replication, you also have ``show_binlog`` and ``show_relaylog`` in every sandbox as a shortcut to display replication logs easily.

## Sandbox customization

There are several ways of changing the default behavior of a sandbox.

1. You can add options to the sandbox being deployed using ``--my-cnf-options="some mysqld directive"``. This option can be used many times. The supplied options are added to my.sandbox.cnf
2. You can specify a my.cnf template (``--my-cnf-file=filename``) instead of defining options line by line. dbdeployer will skip all the options that are needed for the sandbox functioning.
3. You can run SQL statements or SQL files before or after the grants were loaded (``--pre-grants-sql``, ``--pre-grants-sql-file``, etc). You can also use these options to peek into the state of the sandbox and see what is happening at every stage.
4. For more advanced needs, you can look at the templates being used for the deployment, and load your own instead of the original s(``--use-template=TemplateName:FileName``.)

For example:

    $ dbdeployer deploy single 5.6.33 --my-cnf-options="general_log=1" \
        --pre-grants-sql="select host, user, password from mysql.user" \
        --post-grants-sql="select @@general_log"

    $ dbdeployer defaults templates list
    $ dbdeployer defaults templates show templateName > mytemplate.txt
    # edit the template
    $ dbdeployer deploy single --use-template=templateName:mytemplate.txt 5.7.21

dbdeployer will use your template instead of the original.

5. You can also export the templates, edit them, and ask dbdeployer to edit your changes.

Example:

    $ dbdeployer defaults templates export single my_templates
    # Will export all the templates for the "single" group to the direcory my_templates/single
    $ dbdeployer defaults templates export ALL my_templates
    # exports all templates into my_templates, one directory for each group
    # Edit the templates that you want to change. You can also remove the ones that you want to leave untouched.
    $ dbdeployer defaults templates import single my_templates
    # Will import all templates from my_templates/single

Warning: modifying templates may block the regular work of the sandboxes. Use this feature with caution!

6. Finally, you can modify the defaults for the application, using the "defaults" command. You can export the defaults, import them from a modified JSON file, or update a single one on-the-fly.

Here's how:

    $ dbdeployer defaults show
    # Internal values:
```json
{
    "version": "1.5.0",
    "sandbox-home": "$HOME/sandboxes",
    "sandbox-binary": "$HOME/opt/mysql",
    "use-sandbox-catalog": true,
    "master-slave-base-port": 11000,
    "group-replication-base-port": 12000,
    "group-replication-sp-base-port": 13000,
    "fan-in-replication-base-port": 14000,
    "all-masters-replication-base-port": 15000,
    "multiple-base-port": 16000,
    "group-port-delta": 125,
    "mysqlx-port-delta": 10000,
    "master-name": "master",
    "master-abbr": "m",
    "node-prefix": "node",
    "slave-prefix": "slave",
    "slave-abbr": "s",
    "sandbox-prefix": "msb_",
    "master-slave-prefix": "rsandbox_",
    "group-prefix": "group_msb_",
    "group-sp-prefix": "group_sp_msb_",
    "multiple-prefix": "multi_msb_",
    "fan-in-prefix": "fan_in_msb_",
    "all-masters-prefix": "all_masters_msb_",
    "reserved-ports": [
        1186,
        3306,
        33060
    ],
    "timestamp": "Sat May 12 14:37:53 CEST 2018"
 }
```

    $ dbdeployer defaults update master-slave-base-port 15000
    # Updated master-slave-base-port -> "15000"
    # Configuration file: $HOME/.dbdeployer/config.json
```json
{
    "version": "1.5.0",
    "sandbox-home": "$HOME/sandboxes",
    "sandbox-binary": "$HOME/opt/mysql",
    "use-sandbox-catalog": true,
    "master-slave-base-port": 15000,
    "group-replication-base-port": 12000,
    "group-replication-sp-base-port": 13000,
    "fan-in-replication-base-port": 14000,
    "all-masters-replication-base-port": 15000,
    "multiple-base-port": 16000,
    "group-port-delta": 125,
    "mysqlx-port-delta": 10000,
    "master-name": "master",
    "master-abbr": "m",
    "node-prefix": "node",
    "slave-prefix": "slave",
    "slave-abbr": "s",
    "sandbox-prefix": "msb_",
    "master-slave-prefix": "rsandbox_",
    "group-prefix": "group_msb_",
    "group-sp-prefix": "group_sp_msb_",
    "multiple-prefix": "multi_msb_",
    "fan-in-prefix": "fan_in_msb_",
    "all-masters-prefix": "all_masters_msb_",
    "reserved-ports": [
        1186,
        3306,
        33060
    ],
    "timestamp": "Sat May 12 14:37:53 CEST 2018"
}
```

Another way of modifying the defaults, which does not store the new values in dbdeployer's configuration file, is through the ``--defaults`` flag. The above change could be done like this:

    $ dbdeployer --defaults=master-slave-base-port:15000 \
        deploy replication 5.7.21

The difference is that using ``dbdeployer defaults update`` the value is changed permanently for the next commands, or until you run a ``dbdeployer defaults reset``. Using the ``--defaults`` flag, instead, will modify the defaults only for the active command.

## Sandbox management

You can list the available MySQL versions with

    $ dbdeployer versions

Also "available" is a recognized alias for this command.

And you can list which sandboxes were already installed

    $ dbdeployer sandboxes  # Aliases: installed, deployed

The command "usage" shows how to use the scripts that were installed with each sandbox.

    $ dbdeployer usage
    
    	USING A SANDBOX
    
    Change directory to the newly created one (default: $SANDBOX_HOME/msb_VERSION 
    for single sandboxes)
    [ $SANDBOX_HOME = $HOME/sandboxes unless modified with flag --sandbox-home ]
    
    The sandbox directory of the instance you just created contains some handy 
    scripts to manage your server easily and in isolation.
    
    "./start", "./status", "./restart", and "./stop" do what their name suggests. 
    start and restart accept parameters that are eventually passed to the server. 
    e.g.:
    
      ./start --server-id=1001
    
      ./restart --event-scheduler=disabled
    
    "./use" calls the command line client with the appropriate parameters,
    Example:
    
        ./use -BN -e "select @@server_id"
        ./use -u root
    
    "./clear" stops the server and removes everything from the data directory,
    letting you ready to start from scratch. (Warning! It's irreversible!)
    
    "./send_kill" does almost the same as "./stop", as it sends a SIGTERM (-15) kill
    to shut down the server. Additionally, when the regular kill fails, it will
    send an unfriendly SIGKILL (-9) to the unresponsive server.
    
    "./add_option" will add one or more options to my.sandbox.cnf, and restarts the
    server to apply the changes.
    
    "init_db" and "load_grants" are used during the server initialization, and should not be used
    in normal operations. They are nonetheless useful to see which operations were performed
    to set up the server.
    
    "./show_binlog" and "./show_relaylog" will show the latest binary log or relay-log.
    
    "./my" is a prefix script to invoke any command named "my*" from the 
    MySQL /bin directory. It is important to use it rather than the 
    corresponding globally installed tool, because this guarantees 
    that you will be using the tool for the version you have deployed.
    Examples:
    
        ./my sqldump db_name
        ./my sqlbinlog somefile
    
    "./mysqlsh" invokes the mysql shell. Unlike other commands, this one only works
    if mysqlsh was installed, with preference to the binaries found in "basedir".
    This script is created only if the X plugin was enabled (5.7.12+ with --enable-mysqlx
    or 8.0.11+ without --disable-mysqlx)
    
     USING MULTIPLE SERVER SANDBOX
    On a replication sandbox, you have the same commands (run "dbdeployer usage single"), 
    with an "_all" suffix, meaning that you propagate the command to all the members. 
    Then you have "./m" as a shortcut to use the master, "./s1" and "./s2" to access 
    the slaves (and "s3", "s4" ... if you define more).
    
    In group sandboxes without a master slave relationship (group replication and 
    multiple sandboxes) the nodes can be accessed by ./n1, ./n2, ./n3, and so on.
    
    start_all    [options] > starts all nodes
    status_all             > get the status of all nodes
    restart_all  [options] > restarts all nodes
    stop_all               > stops all nodes
    use_all         "SQL"  > runs a SQL statement in all nodes
    use_all_masters "SQL"  > runs a SQL statement in all masters
    use_all_slaves "SQL"   > runs a SQL statement in all slaves
    clear_all              > stops all nodes and removes all data
    m                      > invokes MySQL client in the master
    s1, s2, n1, n2         > invokes MySQL client in slave 1, 2, node 1, 2
    
    The scripts "check_slaves" or "check_nodes" give the status of replication in the sandbox.
    
    

Every sandbox has a file named ``sbdescription.json``, containing important information on the sandbox. It is useful to determine where the binaries come from and on which conditions it was installed.

For example, a description file for a single sandbox would show:

```json
{
    "basedir": "/home/dbuser/opt/mysql/5.7.22",
    "type": "single",
    "version": "5.7.22",
    "port": [
        5722
    ],
    "nodes": 0,
    "node_num": 0,
    "dbdeployer-version": "1.5.0",
    "timestamp": "Sat May 12 14:26:41 CEST 2018",
    "command-line": "dbdeployer deploy single 5.7.22"
}
```

And for replication:

```json
{
    "basedir": "/home/dbuser/opt/mysql/5.7.22",
    "type": "master-slave",
    "version": "5.7.22",
    "port": [
        16745,
        16746,
        16747
    ],
    "nodes": 2,
    "node_num": 0,
    "dbdeployer-version": "1.5.0",
    "timestamp": "Sat May 12 14:27:04 CEST 2018",
    "command-line": "dbdeployer deploy replication 5.7.22 --gtid --concurrent"
}
```

## Sandbox macro operations

You can run a command in several sandboxes at once, using the ``global`` command, which propagates your command to all the installed sandboxes.

    $ dbdeployer global -h 
    This command can propagate the given action through all sandboxes.
    
    Usage:
      dbdeployer global [command]
    
    Examples:
    
    	$ dbdeployer global use "select version()"
    	$ dbdeployer global status
    	$ dbdeployer global stop
    	
    
    Available Commands:
      restart          Restarts all sandboxes
      start            Starts all sandboxes
      status           Shows the status in all sandboxes
      stop             Stops all sandboxes
      test             Tests all sandboxes
      test-replication Tests replication in all sandboxes
      use              Runs a query in all sandboxes
    
    Flags:
      -h, --help   help for global
    
    

The sandboxes can also be deleted, either one by one or all at once:

    $ dbdeployer delete -h 
    Stops the sandbox (and its depending sandboxes, if any), and removes it.
    Warning: this command is irreversible!
    
    Usage:
      dbdeployer delete sandbox_name (or "ALL") [flags]
    
    Aliases:
      delete, remove, destroy
    
    Examples:
    
    	$ dbdeployer delete msb_8_0_4
    	$ dbdeployer delete rsandbox_5_7_21
    
    Flags:
          --concurrent     Runs multiple deletion tasks concurrently.
          --confirm        Requires confirmation.
      -h, --help           help for delete
          --skip-confirm   Skips confirmation with multiple deletions.
    
    

You can lock one or more sandboxes to prevent deletion. Use this command to make the sandbox non-deletable.

    $ dbdeployer admin lock sandbox_name

A locked sandbox will not be deleted, even when running ``dbdeployer delete ALL``.

The lock can also be reverted using

    $ dbdeployer admin unlock sandbox_name

## Compiling dbdeployer

Should you need to compile your own binaries for dbdeployer, follow these steps:

1. Make sure you have go installed in your system, and that the ``$GOPATH`` variable is set.
2. Run ``go get github.com/datacharmer/dbdeployer``.  This will import all the code that is needed to build dbdeployer.
3. Change directory to ``$GOPATH/src/github.com/datacharmer/dbdeployer``.
4. From the folder ``./pflag``, copy the file ``string_slice.go`` to ``$GOPATH/src/github.com/spf13/pflag``.
5. Run ``./build.sh {linux|OSX} 1.6.0``
6. If you need the docs enabled binaries (see the section "Generating additional documentation") run ``MKDOCS=1 ./build.sh {linux|OSX} 1.6.0``

## Generating additional documentation

Between this file and [the API API list](https://github.com/datacharmer/dbdeployer/blob/master/docs/API/API-1.1.md), you have all the existing documentation for dbdeployer.
Should you need additional formats, though, dbdeployer is able to generate them on-the-fly. Tou will need the docs-enabled binaries: in the distribution list, you will find:

* dbdeployer-1.6.0-docs.linux.tar.gz
* dbdeployer-1.6.0-docs.osx.tar.gz
* dbdeployer-1.6.0.linux.tar.gz
* dbdeployer-1.6.0.osx.tar.gz

The executables containing ``-docs`` in their name have the same capabilities of the regular ones, but in addition they can run the *hidden* command ``tree``, with alias ``docs``.

This is the command used to help generating the API documentation. In addition to the API template, the ``tree`` command can produce:

* man pages;
* Markdown documentation;
* Restructured Text pages;
* Command line completion script (see next section).

    $ dbdeployer-docs tree -h
    This command is only used to create API documentation. 
    You can, however, use it to show the command structure at a glance.
    
    Usage:
      dbdeployer tree [flags]
    
    Aliases:
      tree, docs
    
    Flags:
          --api               Writes API template
          --bash-completion   creates bash-completion file
      -h, --help              help for tree
          --man-pages         Writes man pages
          --markdown-pages    Writes Markdown docs
          --rst-pages         Writes Restructured Text docs
          --show-hidden       Shows also hidden commands
    
    

## Command line completion

There is a file ``./docs/dbdeployer_completion.sh``, which is automatically generated with dbdeployer API documentation. If you want to use bash completion on the command line, copy the file to the bash completion directory. For example:

    # Linux
    $ sudo cp ./docs/dbdeployer_completion.sh /etc/bash_completion.d
    $ source /etc/bash_completion

    # OSX
    $ sudo cp ./docs/dbdeployer_completion.sh /usr/local/etc/bash_completion.d
    $ source /usr/local/etc/bash_completion

Then, you can use completion as follows:

    $ dbdeployer [tab]
        admin  defaults  delete  deploy  global  sandboxes  unpack  usage  versions
    $ dbdeployer dep[tab]
    $ dbdeployer deploy [tab][tab]
        multiple     replication  single
    $ dbdeployer deploy s[tab]
    $ dbdeployer deploy single --b[tab][tab]
        --base-port=     --bind-address=

## Using dbdeployer source for other projects

If you need to create sandboxes from other Go apps, see  [dbdeployer-as-library.md](https://github.com/datacharmer/dbdeployer/blob/master/docs/coding/dbdeployer-as-library.md).

## Semantic versioning

As of version 1.0.0, dbdeployer adheres to the principles of [semantic versioning](https://semver.org/). A version number is made of Major, Minor, and Revision. When changes are applied, the following happens:

* Backward-compatible bug fixes increment the **Revision** number.
* Backward-compatible new features increment the **Minor** number.
* Backward incompatible changes (either features or bug fixes that break compatibility with the API) increment the **Major** number.

The starting API is defined in [API-1.0.md](https://github.com/datacharmer/dbdeployer/blob/master/docs/API/API-1.0.md) (generated manually.)
The file [API-1.1.md](https://github.com/datacharmer/dbdeployer/blob/master/docs/API/API-1.1.md) contains the same API definition, but was generated automatically and can be used to better compare the initial API with further version.


