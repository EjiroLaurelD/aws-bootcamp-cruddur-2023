# Week 4 — Postgres and RDS
- I created an rds instance with the following code
```
aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username root \
  --master-user-password Dbpassword1 \
  --allocated-storage 20 \
  --availability-zone us-east-1a \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection
  ```
![rds](./assets/rds.png)

I connected to postgress locally using the command 
`psql -Upostgres --host localhost`
I created a table for cruddur
`CREATE database cruddur;`
![db](./assets/cruddur-table.png)

I created schema.sql file in a db directory I created in the backend

I added UUID Extention by adding this code to my schema.sql file
`CREATE EXTENSION IF NOT EXISTS "uuid-ossp";`
then imported the command with the following code
`psql cruddur < db/schema.sql -h localhost -U postgres`

I made the connection url for my psql an environment variable in my schema file with the code
`export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`
`gp env CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`
I also set the environment variable for production url on gitpod

I created a bin directory and made 3 files in it 
db_create db_drop and db_shcema_load then i dropped, created and loaded my schema tabe with the commands `./db_drop` `./db_create` `./db-schema_load`
![sql](./assets/sql.png)


I created extra files in the directory, db_connect which i used to connect directly to the database to view my tables using the psql command `\dt`
![db](./assets/sql-connect.png)

I created the file db_seed to popuulate my table, then db-sessions to start sessions in production mode or otherwise



I got my gitpod ip with the command `GITPOD_IP=$(curl ifconfig.me)
`
I set the environment variables for my security group id and group rule id
`export DB_SG_ID="redacted"`
`gp env DB_SG_ID="redacted"`

`EXPORT DB_SG_RULE_ID="redacted"`
`gp env DB_SG_RULE_ID="redacted"`

Whenever I needed to update my security groups i use this code for access.

```
aws ec2 modify-security-group-rules \
    --group-id $DB_SG_ID \
    --security-group-rules "SecurityGroupRuleId=$DB_SG_RULE_ID,SecurityGroupRule={Description=GITPOD,IpProtocol=tcp,FromPort=5432,ToPort=5432,CidrIpv4=$GITPOD_IP/32}"
```
