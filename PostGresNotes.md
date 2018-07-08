# Postgres SQL

## Installing Postgres
1. Use a package manager (apt-get for Ubuntu or Homebrew for OS X)
    * `brew intall postgresql`
2. Install pg gem to interact with Postgres from Ruby code
    * `gem install pg`
3. Start the postgressql server
    * `brew services start postgresql`
    * If you want it to start up at computer boot, run: `pg_ctl -D /usr/local/var/postgres start && brew services start postgresql`

## Configuring Postgres
1. Use the __psql__ utility installed with Postgres that allows the carrying out of administrative functions without SQL commands.
    * In CL, `psql postgres`, and you'll see `postgres=#`
    * To quit, `\q`
    * To see what users are installed, type `\du`

### Creating Users
1. Postgres sets up a default user, but it is a bad idea to use them for anything except local development because they are super user accounts.
2. Create new users and roles either by
    * Executing the __CREATE ROLE__ SQL query
    * Using the __createuser__ utility that comes with Postgres
3. __CREATE ROLE__ - `CREATE ROLE username WITH LOGIN PASSWORD 'quoted password' [OPTIONS];`
    * This creates a user without any permissions. To add permissions, do:
        * `ALTER ROLE username PERMISSION`
4. __createuser__ - instead of logging into psql to execute SQL queries, use a command line interface to do the sme task
|Keyword|Task|
|-------|----|
|createuser|creates a user|
|createdb|creates a database|
|dropuser|deletes a user|
|dropdb|deletes a database|
|postgres|executes the SQL server itself|
|pg_dump|dumps contents of a single database to a file|
|pg_dumpall|dumps all databases to a file|
|psql|keyword to execute psql|
* Create a user `createuser username`

## Create new rails API with PostgreSQL database
* `rails new super-awesome-api --api --database=postgresql`

## Creating a Database
1. There are two ways
    * Executing SQL commands with psql
    * createdb command line utility
2. Creating with psql
    * `psql postgres -U username`
    * `CREATE DATABASE databasename;`
    * Add at least one user who has permission to access the database.
        * `GRANT ALL PRIVILEGES ON DATABASE databasename TO username;`
        * `\list`
            * This lists all the databases in postgres
        * `\connect databasename`
            * Connects to specific database
        * `\dt`
            * Lists the tables in the currently connected database
3. Creating with createdb utility
    * Creating the database is simpler, but cannot manage db after creation.
    * `createdb databasename -U username`
    * This creates the database and passes the user to use for connecting to the datavase.
    * To change the database name, must use psql
        * `psql postgres -U username`
        * `ALTER DATABASE databasename RENAME TO anothername;`

## Connecting Ruby to Postgres
1. If `rails new myapp --database=postgresql` wasn't run, then the rails app is automatically using sqlite3, and has to be altered.
2. In __/config/database.yml__ configure the database
```
development:
  adapter: postgresql
  encoding: unicode
  database: myapp_development
  pool: 5
  username: myapp
  password: password1

test:
  adapter: postgresql
  encoding: unicode
  database: myapp_test
  pool: 5
  username: myapp
  password: password1
```
3. Alter the gemfile, to include pg `gem pg` and remove the sqlite3
4. Run `rake db:setup`
    * This creates development and test databases, and schema_migrations tables in each
5. Create a scaffold of your tables with
    * `rails g scaffold Post title:string body:text`
    * `rake db:migrate`


## References
[1](https://www.codementor.io/engineerapart/getting-started-with-postgresql-on-mac-osx-are8jcopb)
[2](https://www.digitalocean.com/community/tutorials/how-to-setup-ruby-on-rails-with-postgres)