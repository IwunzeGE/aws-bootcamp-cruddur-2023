# Week 4 — Postgres and RDS

Although we'd be launching the RDS instance from the CLI, Lets take a walk on how to use the console too

![rds1](https://user-images.githubusercontent.com/110903886/227605795-55fca5b3-1d1f-43f8-9bb5-85e021fecf6f.png)

![rds2](https://user-images.githubusercontent.com/110903886/227605867-78c84c29-efa1-481a-a046-5cd0d13543d3.png)

![rds3](https://user-images.githubusercontent.com/110903886/227605897-6d9c32eb-30c0-4cd7-8db2-4e3f28125fea.png)

- Select the template - We'd use free tier although it restricts us access to some config options
- Setup database-1
- Set up password if you're not managing the credentials with AWS Secrets Manager. 
**Note: AWS Secrets Manager cost $1 to use.**

- Choose the right instance configuration that fits your need. Although we'd be limited to the db.t3.micro and db.t4g.micro free tier instance.

![rds instance config](https://user-images.githubusercontent.com/110903886/227606167-bad6e2b9-d292-4072-be25-db28138bb0f4.png)

- Allocate storage and enable autoscalinng
- Under the connectivity section, we'd allow public access although it's not the best in terms of secuirity. But there's always a trade cost with secuirity. Despite being public, the secuirity groups would serve as a check medium.

![rds4](https://user-images.githubusercontent.com/110903886/227606333-93ab1962-e542-4b1c-ad4d-1acb279087bd.png)

- With AWS it's okay to use the default vpc but with some other cloud provider, we might want to use another one.

![rds5](https://user-images.githubusercontent.com/110903886/227607338-28b071c5-1866-4ae1-ba12-1ed157402544.png)

- It is important to enable automated backups in production and enable deletion protection

![rds6](https://user-images.githubusercontent.com/110903886/227607602-e7338550-030a-429d-803c-fe40060445a6.png)
 

Now, lets provisioning an RDS instance with db named 'cruddur' from the CLI by running the code below.

```
aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username root \
  --master-user-password huEE33z2Qvl383 \
  --allocated-storage 20 \
  --availability-zone ca-central-1a \
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
![cli code](https://user-images.githubusercontent.com/110903886/227614009-b1b3b1ef-3544-46f8-b133-8d28324fb514.png)

- Confirm from the console to see if it worked.

![cli code check](https://user-images.githubusercontent.com/110903886/227614048-f8024b3f-98b3-42f0-8f97-7b6a98c2abb3.png)

Connect to psql via the psql client cli tool remember to use the host flag to specific localhost
`psql -U postgres --host localhost`

![login postgres](https://user-images.githubusercontent.com/110903886/227617142-5c051ef5-66e9-4f22-bfa3-bcf6a9e213c7.png)


### Common PSQL commands:

```
\x on -- expanded display when looking at data
\q -- Quit PSQL
\l -- List all databases
\c database_name -- Connect to a specific database
\dt -- List all tables in the current database
\d table_name -- Describe a specific table
\du -- List all users and their roles
\dn -- List all schemas in the current database
CREATE DATABASE database_name; -- Create a new database
DROP DATABASE database_name; -- Delete a database
CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...); -- Create a new table
DROP TABLE table_name; -- Delete a table
SELECT column1, column2, ... FROM table_name WHERE condition; -- Select data from a table
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); -- Insert data into a table
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition; -- Update data in a table
DELETE FROM table_name WHERE condition; -- Delete data from a table
```

- Create a database named `cruddur`

![create db](https://user-images.githubusercontent.com/110903886/227622173-7d2d4604-fc85-45ad-b816-03b86f090234.png)

### Import Script

We'll create a new SQL file called `schema.sql` and we'll place it in `backend-flask/db`

Paste the below code into `schema.sql`. This ia aimed at adding UUID extension. We are going to have Postgres generate out UUIDs. We'll need to use an extension called:

```
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

The command to import:

`psql cruddur < backend-flask/db/schema.sql -h localhost -U postgres`

![create extension](https://user-images.githubusercontent.com/110903886/227621858-52492388-b7a2-43ea-a8fc-9d302f321681.png)

To make connecting to psql client easier, lets export our postgres credentials as an env

`export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"`

Try connecting using `psql $CONNECTION_URL`

![connectionurl](https://user-images.githubusercontent.com/110903886/227625339-e172ca14-0636-43c1-96b6-41dcc2800ebd.png)


In the `backend-flask`, create a folder named `bin` with three files in it 	`db-create`, `db-drop` and `db-schema-load`

![create bin](https://user-images.githubusercontent.com/110903886/227659460-126c2995-14fa-40c9-9143-7339473652d4.png)

- Change the permissions of the files and make them executable.

![chmod for bin](https://user-images.githubusercontent.com/110903886/227659459-524b6571-386f-428a-a1f1-5151dca8a4e7.png)


- Edit the `db-drop`

```
#! /usr/bin/bash

echo db-drop

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "DROP  DATABASE cruddur;"
```

- Run it using `./bin/db-drop`

![db-drop](https://user-images.githubusercontent.com/110903886/227660780-2979765f-84dc-426c-8409-4e2bc41d7be5.png)

- Edit the `db-create`

```
#! /usr/bin/bash

echo db-create

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "CREATE DATABASE cruddur;"
```

![db-create](https://user-images.githubusercontent.com/110903886/227661022-f0385b1a-f853-4ae3-bca9-f4194e07fa37.png)

- Edit the `db-schema-load`

```
#! /usr/bin/bash

schema_path="$(realpath .)/db/schema.sql"

echo $schema_path

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL cruddur < $schema_path
```

![db-schema](https://user-images.githubusercontent.com/110903886/227662499-28f30d54-d143-49aa-a2bc-505c1f04931e.png)


- Run in production mode, replace the `db-schema-load` with the code below

```
#! /usr/bin/bash

schema_path="$(realpath .)/db/schema.sql"

echo $schema_path

if [ "$1" = "prod" ]; then
  echo "Running using production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $schema_path
```

![load prod](https://user-images.githubusercontent.com/110903886/227928667-2f24b898-af39-4f0d-b1f3-9daa9397bbae.png)


- Add colouring to the bash code in `db-schema-load` to make prints nicer. The new code becomes

```
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-schema-load"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

schema_path="$(realpath .)/db/schema.sql"

echo $schema_path

if [ "$1" = "prod" ]; then
  echo "Running using production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $schema_path
```
![color](https://user-images.githubusercontent.com/110903886/227928590-aa8f0e4e-975a-440d-a7fd-d58f2973eabb.png)

## Create our tables
[Documentation on creating tables](https://www.postgresql.org/docs/current/sql-createtable.html)

In the `schema.sql` file, paste the below code. This will first delete any similar existing table and proceed to creating new tables.

```
DROP TABLE IF EXISTS public.users;
DROP TABLE IF EXISTS public.activities;


CREATE TABLE public.users (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  display_name text,
  handle text
  cognito_user_id text,
  created_at TIMESTAMP default current_timestamp NOT NULL
);


CREATE TABLE public.activities (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  message text NOT NULL,
  replies_count integer DEFAULT 0,
  reposts_count integer DEFAULT 0,
  likes_count integer DEFAULT 0,
  reply_to_activity_uuid integer,
  expires_at TIMESTAMP,
  created_at TIMESTAMP default current_timestamp NOT NULL
);
```

![create table](https://user-images.githubusercontent.com/110903886/227932318-bfed3506-ac55-48df-9b8d-d7bc69536fc7.png)

![tablesssss](https://user-images.githubusercontent.com/110903886/227934785-84faa1a6-1175-40b5-ad36-1f76b6e49241.png)


- Create a `bin/db-connect` file in the `backend-flask` dir, make it executable and paste the code below

```
#! /usr/bin/bash

psql $CONNECTION_URL
```
![db-connect](https://user-images.githubusercontent.com/110903886/227934839-2696d97b-1931-4cf8-9277-10641fb924f3.png)


- Create a `bin/db-seed`, make it executable  and `db/seed.sql` file in the `backend-flask` dir, and paste the code below respectively

```
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-seed"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

seed_path="$(realpath .)/db/seed.sql"

echo $seed_path

if [ "$1" = "prod" ]; then
  echo "Running using production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $seed_path
```

```
-- this file was manually created
INSERT INTO public.users (display_name, handle, cognito_user_id)
VALUES
  ('Andrew Brown', 'andrewbrown' ,'MOCK'),
  ('Iwunze Godspower', 'ekiwunze' ,'MOCK');

INSERT INTO public.activities (user_uuid, message, expires_at)
VALUES
  (
    (SELECT uuid from public.users WHERE users.handle = 'andrewbrown' LIMIT 1),
    'This was imported as seed data!',
    current_timestamp + interval '10 day'
  )
```
 
![dbseed](https://user-images.githubusercontent.com/110903886/227949967-7b68801d-e278-41b4-90c6-f6da5db51059.png)


























