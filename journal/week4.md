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
I created database cruddur
`CREATE database cruddur;`
I created schema.sql file in a db directory i created in the backend

I added UUID Extention by adding this code to my schema.sql file
`CREATE EXTENSION IF NOT EXISTS "uuid-ossp";`
then imported the command with the following code
`psql cruddur < db/schema.sql -h localhost -U postgres`

I made the connection url for my psql an environment variable in my schema file with the code
`export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`
`gp env CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`
I also set the environment variable for production url on gitpod

I created a bin directory and made 3 files in it 
db_create db_drop and db_shcema_load then i dropped my database using the command `./db-drop`

I created extra files in the directory, db_connect which i used to connect directly to the database to view my tables using the psql command `\dt`
![db](./assets/db.png)
