This container is based on the [Fedora](../fedora/23) base container.

The Docker file uses the distribution package manager to install the [PostgreSQL](http://www.postgresql.org/) database server and client.

Running the container with no arguments will create a new database, with random database name, user name and password.

```
    #
    # Run a container in the foreground. 
    docker run \
       'cosmopterix/pgsql'

    Checking database directory [/var/lib/pgsql]
    Updating database directory [/var/lib/pgsql]
    Checking socket directory [/var/lib/pgsql]
    Checking for database data [postgres]
    Creating database data [postgres]
    ....
    ....
    Checking database user [neDei3iewa]
    Creating database user [neDei3iewa]
    CREATE ROLE
    Checking database data [Athie6uJ0i]
    Creating database data [Athie6uJ0i]
    CREATE DATABASE
    ....
    ....
    Initialization process complete.
    Starting database service
    

When running in the foreground, you can use `Ctrl+C` to stop the container.

The Docker `--detach` option will run the container in the background.

```
    #
    # Run a container in the background. 
    docker run \
        --detach \
       'cosmopterix/pgsql'
```

The Docker `ps` command will list running containers.

```
    #
    # List the active containers.
    docker ps

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    9b9635dd4587        cosmopterix/pgsql   "/usr/local/bin/entry"   3 seconds ago       Up 2 seconds        5432/tcp            hungry_aryabhata
```

When running in the background, you need to use use the Docker `stop` command with either the container id or name to stop the container.

```
    #
    # Stop an active container.
    docker stop 9b9635dd4587
```

Naming the container makes it easier to refer to it in subsequent commands.

```
    #
    # Run a named container in the background.
    docker run \
        --detach \
        --name 'albert' \
       'cosmopterix/pgsql'
```

```
    #
    # List the active containers.
    docker ps

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    668c480863b2        cosmopterix/pgsql   "/usr/local/bin/entry"   3 seconds ago       Up 2 seconds        5432/tcp            albert

To use the same name again you need to stop and then remove the container. 

```
    #
    # Stop an active container.
    docker stop albert

    #
    # Remove a container.
    docker rm albert
```

The stop and remove commands can be nested together as a single command using `$()` .

```
    #
    # Stop and remove the container.
    docker rm $(docker stop 'albert')
```

The Docker `exec` command can be used to connect to a running container and run
another program, for example the following command will start a bash shell
inside the container.

```
    #
    # Run a bash shell in a running container.
    docker exec \
        --tty \
        --interactive \
        'albert' \
        'bash'

        ls -al

        exit
```

The container entrypoint script saves deatils of the database configuration in a `/database.save` file inside the container.

You can use the Docker `exec` command to connect to the container and read the `/database.save` config file.

```
    #
    # Display the contents of /database.save in the container.
    docker exec \
        --tty \
        --interactive \
        'albert' \
        cat '/database.save'

    #
    # Admin settings
    admindata=postgres
    adminuser=postgres
    adminpass=ahdues2Aad

    ....
    
    #
    # Database settings
    databasename=Kie5aeQuai
    databaseuser=IevoB8om8o
    databasepass=ighei0Ieyi

```

The entry point script checks for a `/database.config` script file
at startup. If the config file is found it is executed using the
bash `source` command.

This provides a method for setting environment variables at the
begining of the initialization process which can then be used  by
the rest of the entrypoint script.

* The entry point script uses `adminuser` and `adminpass` environment
variables to configure the server admin account.
* The entry point script uses `databasename` `databaseuser` and `databasepass`
environment variables to configure the new database.
* If the user names and passwords are not specified then random default
values are generated.

```
    #
    # Create a temp file.
    tempcfg=$(mktemp)
    
    #
    # Write to our database config.
    cat > "${tempcfg:?}" << EOF
adminuser=helen
adminpass=$(pwgen 10 1)
databasename=testdb
databaseuser=stephany
databasepass=$(pwgen 10 1)
EOF

    #
    # Run a new container with ${tempcfg} mounted
    # as /database.config inside the container
    docker run \
        --detach \
        --name 'albert' \
        --volume "${tempcfg}:/database.config" \
       'cosmopterix/pgsql'

```

In this container, the adminuser will be set to `helen`, and the 
database name and user name will be `testdb` and `stephany`.

```
    #
    # Display the contents of /database.save in the container.
    docker exec \
        --tty \
        --interactive \
        'albert' \
        cat '/database.save'

    #
    # Admin settings
    admindata=postgres
    adminuser=helen
    adminpass=aingo2aiY4

    ....

    #
    # Database settings
    databasename=testdb
    databaseuser=stephany
    databasepass=ahTahbi3zo

```

The entry point script also checks for `.sh`, `.sql` or `.sql.gz` files
in the `/database.init/` directory inside the container.

* Shell script, `*.sh`, files will be executed inside the container using the database server login.
* SQL, `*.sql`, files will be run on the new database using the `psql` command line client.
* Gzipped, `*.sql.gz`, files will be unzipped and then run on the new database using the `psql` command line client.

You can use the Docker `--volume` option to mount a local directory as `/database.init/` inside the container.

```

    #
    # Create a temp directory.
    tempdir=$(mktemp -d)
    
    #
    # Copy our SQL scripts into the temp directory
    cp "pgsql/sql/alpha-source.sql" "${tempdir}/001.sql"
    cp "data/alpha-source-data.sql" "${tempdir}/002.sql"

    #
    # Run our database container with ${tempdir} mounted
    # as /database.init/ inside the container
    docker run \
        --detach \
        --name 'albert' \
        --volume "${tempdir}:/database.init/" \
       'cosmopterix/pgsql'

```

In this container, the new database will be initialized with the SQL commands
from the `alpha-source.sql` and `alpha-source-data.sql` SQL files.

```
    docker logs \
        --follow \
        'albert'

    ....
    ....
    Running local instance
    ....
    Checking database user [sei5aijeiL]
    Creating database user [sei5aijeiL]
    CREATE ROLE
    Checking database data [Li4dih1lei]
    Creating database data [Li4dih1lei]
    CREATE DATABASE

    Checking init directory [/database.init]

    Running init scripts
    /usr/local/bin/entrypoint: running [/database.init/001.sql]
    CREATE TABLE

    /usr/local/bin/entrypoint: running [/database.init/002.sql]
    INSERT 0 1
    INSERT 0 1
    INSERT 0 1
    ....

```

The container image also includes a startup script for the the`psql` commandline client.

Using the Docker `exec` command to run `psql-client` will launch the psql commandline client and automatically connect it to the new database.

```
    docker exec \
        --tty \
        --interactive \
        'albert' \
        'psql-client'

        \dt

        \q

```

Combining all of the above, we can create a database with
specific username and passwords, initialise the database with
data from our SQL scripts, and then login
and run our tests.

```
    #
    # Create our config file.
    tempcfg=$(mktemp)
    cat > "${tempcfg:?}" << EOF
databasename=testdb
databaseuser=stephany
databasepass=$(pwgen 10 1)
EOF

    #
    # Create our scripts directory.
    tempdir=$(mktemp -d)
    cp "mysql/sql/alpha-source.sql" "${tempdir}/001.sql"
    cp "data/alpha-source-data.sql" "${tempdir}/002.sql"


    #
    # Run our database container with ${tempcfg} mounted
    # as /database.config inside the container
    docker run \
        --detach \
        --name 'albert' \
        --volume "${tempdir}:/database.init/" \
        --volume "${tempcfg}:/database.config" \
       'cosmopterix/pgsql'


    #
    # Login and run a test.
    docker exec \
        --tty \
        --interactive \
        'albert' \
        'psql-client'

            \pset pager off 
            
            SELECT id, ra, decl FROM alpha_source ;
            SELECT id, ra, decl FROM alpha_source LIMIT 10 ;
            SELECT id, ra, decl FROM alpha_source OFFSET 10 ;
            SELECT id, ra, decl FROM alpha_source LIMIT 10 OFFSET 10 ;

            \q

```

