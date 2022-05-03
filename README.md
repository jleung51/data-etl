# Foundations: Data Pipeline

Data ETL pipeline to clean, process, and aggregate data from Canadian housing starts.

Built with [Apache Airflow](https://airflow.apache.org/), [dbt](https://www.getdbt.com/), and [Amazon Web Services EC2](https://aws.amazon.com/ec2/).

___

## Setup Instructions

### Set up AWS EC2 (Host for the Database and Orchestrator)

#### Provision an EC2 Instance

1. Navigate to [Amazon Web Services](https://aws.amazon.com/) and create an account.
1. In the search bar at the top, search for and click **EC2**.
1. Click the **Launch Instance** button and follow the instructions. For this example, we will be using the Amazon Linux AMI operating system.
1. Download the `.pem` key file and keep it secure.
1. Wait until the instance state is **Running**.

#### Connect to the EC2 Instance using AWS CloudShell

1. In the AWS EC2 service, in the left sidebar, select **Instances**.
1. Select the newly created instance.
1. At the top of the screen, click **Connect**.
1. Choose any of the provided options to connect to the instance.

As an alternative to CloudShell, you can also use SSH from your local computer.

### Set up PostgreSQL (Database)

Log into the EC2 instance using AWS CloudShell or SSH.

Check the PostgreSQL packages available from the Amazon Linux package manager:
```bash
sudo amazon-linux-extras list | grep postgres
```

Choose a version of PostgreSQL and install it:
```bash
sudo amazon-linux-extras install postgresql13
sudo yum clean metadata
sudo yum install postgresql postgresql-server
```

Verify that the package installation was successful:
```bash
psql --version
```

Initialize the database configuration:
```bash
sudo /bin/postgresql-setup initdb
```

Enable the PostgreSQL system service:
```bash
sudo systemctl enable --now postgresql
```

Verify that the service is running:
```bash
systemctl status postgresql
```

Alternatively, follow the instructions for your operating system under the [PostgreSQL Downloads page](https://www.postgresql.org/download/).


#### Log into PostgreSQL as admin

Change to the `postgres` Linux user:
```bash
sudo su - postgres
```

From here, you can log into the PostgreSQL shell (`psql`) as the database user `postgres`:
```bash
psql
```

You can connect to a database manually while in the `psql` shell:
```sql
# List databases
\list

# Connect to a specific database
\connect <DATABASE_NAME>
```


To exit psql, type:
```bash
exit
```

To return to the original Linux user at any time, type:
```bash
exit
```

#### Initiate a User and Database

If not already in the `postgres` Linux user, change to it.

Change the password field in the snippet below and create a PostgreSQL admin user with root access:
```bash
psql -c "CREATE USER admin WITH ENCRYPTED PASSWORD '<PASSWORD>';"
psql -c "GRANT postgres TO admin;"

psql -c "CREATE USER airflow WITH ENCRYPTED PASSWORD '<PASSWORD>';"
```

Create a database using the `postgres` `psql` user:
```bash
sudo -u postgres createdb foundations
```


#### Enable Password Authentication for the new User

If not already in the `postgres` Linux user, change to it.

Print the location of the configuration file:
```bash
psql -c "SHOW hba_file;"
```

Open the file:
```bash
edit /<PATH>/pg_hba.conf
```

Add a new line which says:
```
local foundations airflow    ident
```

Restart PostgreSQL:
```bash
sudo service postgresql restart
```


### Airflow (Orchestrator)

Instructions under construction.


#### Connect Airflow with PostgreSQL

These steps are detailed instructions built from Airflow's [Setting up a PostgreSQL Database guide](https://airflow.apache.org/docs/apache-airflow/2.2.2/howto/set-up-database.html#setting-up-a-postgresql-database).


Using the `psql` admin, run the SQL script `setup/psql-airflow.sql` to generate the Airflow database and user:
```bash
sudo -u postgres psql -d <DATABASE_NAME> -f ./setup/psql-airflow.sql --echo-all
```

By default, this will create the following:
- Database: `airflow_db`
- PostgreSQL user (with full access to `airflow_db`): `airflow_user`
- PostgreSQL password: `airflow_pass`


Edit the Airflow configuration file (located at `~/airflow/airflow.cfg` by default) and modify the value of `sql_alchemy_conn` as follows:
```
sql_alchemy_conn = postgresql+psycopg2://airflow_user:airflow_pass@<HOST>/airflow_db
```

**Note**: If Airflow and PostgreSQL are on the same machine, then `<HOST>` can be replaced with `localhost`; otherwise, use the hostname of PostgreSQL.


Install the Python library `psycopg2` to use the connector:
```bash
pip3 install psycopg2
```


Initiate the Airflow database:
```bash
airflow db init
```

Start up Airflow and it will begin to use PostgreSQL as the database.

