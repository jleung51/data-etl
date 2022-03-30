# Foundations: Data Pipeline

Data ETL pipeline to clean, process, and aggregate data from Canadian housing starts.

Built with [Apache Airflow](https://airflow.apache.org/), [dbt](https://www.getdbt.com/), and [Amazon Web Services EC2](https://aws.amazon.com/ec2/).

___

## Setup Instructions

### Set up AWS EC2 (Host for the Database and Orchestrator)

#### Provision an EC2 Instance
Navigate to [Amazon Web Services](https://aws.amazon.com/) and create an account.
In the search bar at the top, search for and click **EC2**.
Click the **Launch Instance** button and follow the instructions. For this example, we will be using the Amazon Linux AMI operating system.
Download the `.pem` key file and keep it secure.
Wait until the instance state is **Running**.

#### Connect to the EC2 Instance using AWS CloudShell

In the AWS EC2 service, in the left sidebar, select **Instances**.
Select the newly created instance.
At the top of the screen, click **Connect**.
Choose any of the provided options to connect to the instance.

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
```

Verify that the installation was successful:
```bash
psql --version
```

Alternatively, follow the instructions for your operating system under the [PostgreSQL Downloads page](https://www.postgresql.org/download/).


#### Allow Password Authentication

Modify the PostgreSQL configuration to allow password identification. First, open the file:
```bash
sudo edit /etc/postgresql/<POSTGRES_VER>/main/pg_hba.conf
```

Edit the line `local all postgres peer` by replacing `peer` with `ident`.

Restart PostgreSQL:
```bash
sudo service postgresql restart
```

#### Create a PostgreSQL User

Using the `postgres` user, open the `psql` shell:
```bash
sudo -u postgres psql
```

In the `psql` shell, run the following commands to create a user with root access:

```sql
CREATE USER <USERNAME>
WITH ENCRYPTED PASSWORD 'password';

GRANT ROOT TO <USERNAME>;
```

Exit the shell:
```bash
exit
```


Using the `postgres` user, create a database:
```bash
sudo -u postgres createdb <DATABASE_NAME>
```

#### Log In

You should now be able to log in with the following command:
```bash
psql -U <USERNAME>
```

Alternatively, connect to a database while logging in:
```bash
psql -U <USERNAME> <DATABASE_NAME>
```

Connect to a database manually while in the `psql` shell:
```sql
# List databases
\list

# Connect to a specific database
\connect <DATABASE_NAME>
```

You can also use the `postgres` user to log in as the `postgres` admin user:
```bash
sudo -u postgres psql
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

