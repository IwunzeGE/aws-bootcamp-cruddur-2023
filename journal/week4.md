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


## See what connections we are using
- Create a file `/bin/db-sessions` and paste the code below

```
#! /usr/bin/bash

#echo "== db-sessions"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-sessions"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "select pid as process_id, \
       usename as user,  \
       datname as db, \
       client_addr, \
       application_name as app,\
       state \
from pg_stat_activity;"
```

![db-sessions](https://user-images.githubusercontent.com/110903886/227956978-8c4089d7-fb19-41d3-a7e3-a0efa9d9b379.png)


- Create a `/bin/db-setup` file that runs the other bin files to give flexibility and help make it easier to maintain.

![db-setup](https://user-images.githubusercontent.com/110903886/228345451-6a7c99d2-68d5-4e1f-8626-f6ad0324e403.png)

## DB Object and Connection Pool


- Let's install the postgres driver. We'll add the following to our `requirments.txt` and install it

```
psycopg[binary]
psycopg[pool]
```

`pip install -r requirements.txt`

- Create a file `db.py` in the `lib` dir and add the following code

```
from psycopg_pool import ConnectionPool
import os

def query_wrap_object(template):
  sql = f"""
  (SELECT COALESCE(row_to_json(object_row),'{{}}'::json) FROM (
  {template}
  ) object_row);
  """
  return sql

def query_wrap_array(template):
  sql = f"""
  (SELECT COALESCE(array_to_json(array_agg(row_to_json(array_row))),'[]'::json) FROM (
  {template}
  ) array_row);
  """
  return sql

connection_url = os.getenv("CONNECTION_URL")
pool = ConnectionPool(connection_url)
```


- Update the backend in Dockercompose file with the connection url
```
CONNECTION_URL: "${CONNECTION_URL}"
```

- Edit `home_activities.py`. It should now look like this


```
from datetime import datetime, timedelta, timezone
from opentelemetry import trace
from lib.db import pool, query_wrap_object, query_wrap_array

tracer = trace.get_tracer("home.activities")

class HomeActivities:
  def run(cognito_user_id=None):
    print("HOME ACTIVITY")
    #logger.info("HomeActivities")
    with tracer.start_as_current_span("home-activites-mock-data"):
      span = trace.get_current_span()
      now = datetime.now(timezone.utc).astimezone()
      span.set_attribute("app.now", now.isoformat())

      sql = query_wrap_array("""
      SELECT
        activities.uuid,
        users.display_name,
        users.handle,
        activities.message,
        activities.replies_count,
        activities.reposts_count,
        activities.likes_count,
        activities.reply_to_activity_uuid,
        activities.expires_at,
        activities.created_at
      FROM public.activities
      LEFT JOIN public.users ON users.uuid = activities.user_uuid
      ORDER BY activities.created_at DESC
      """)
      print(sql)
      with pool.connection() as conn:
        with conn.cursor() as cur:
          cur.execute(sql)
          # this will return a tuple
          # the first field being the data
          json = cur.fetchone()
      return json[0]
```

We should have succesfully implemented our first query

![checkkkkkkkk](https://user-images.githubusercontent.com/110903886/229310949-be875ff3-f2a4-4617-9d11-37551f3a72e0.png)


## Connecting with RDS

- Set our RDS connection link as $PROD_URL_CONNECTION

```
PROD_CONNECTION_URL=postgresql://cruddurroot:Password@cruddur-db-instance.ccvmxgrrnpgz.us-east-1.rds.amazonaws.com:5432/cruddur
```

- Let's update our Secuirity group from the AWS console to allow connection from our gitpod IP.

![inbound rules](https://user-images.githubusercontent.com/110903886/229311135-d15f57ed-ef2b-41a4-8e3d-ec18ac52b94f.png)

- To get our gitpod IP address,

```
curl ifconfig.me
```

- Set it as an env

```
GITPOD_IP=$(curl ifconfig.me)
```
![IP](https://user-images.githubusercontent.com/110903886/229311222-915f1f86-5487-4487-bc27-e4965861bac9.png)

- Connect using

```
psql $PROD_URL_CONNECTION
```

![rds connnect](https://user-images.githubusercontent.com/110903886/229311272-b98a2886-21da-4c8b-ba7c-4a98b5d122c7.png)

![ll](https://user-images.githubusercontent.com/110903886/229311402-3b36b13a-ef2a-4dbf-aed6-cc235db4917c.png)

We'll create an inbound rule for Postgres (5432) and provide the GITPOD ID.

We'll get the security group rule id so we can easily modify it in the future from the terminal here in Gitpod.

```
export DB_SG_ID="sg-092eec5941487c33c"
gp env DB_SG_ID="sg-092eec5941487c33c"

export DB_SG_RULE_ID="sgr-0df0ff668f174dda1"
gp env DB_SG_RULE_ID="sgr-0df0ff668f174dda1"
```

Whenever we need to update our security groups we can do this for access.

```
aws ec2 modify-security-group-rules \
    --group-id $DB_SG_ID \
    --security-group-rules "SecurityGroupRuleId=$DB_SG_RULE_ID,SecurityGroupRule={Description=GITPOD,IpProtocol=tcp,FromPort=5432,ToPort=5432,CidrIpv4=$GITPOD_IP/32}"
```

![test1](https://user-images.githubusercontent.com/110903886/229312714-3a3929b3-79ec-4cd5-8131-aefd91c05388.png)
![test2](https://user-images.githubusercontent.com/110903886/229312716-c098d64a-2951-4b63-82b4-f228d2b3eddf.png)
![test3](https://user-images.githubusercontent.com/110903886/229312713-69e02a59-3b0a-4ab4-b681-25275f690fb8.png)

- Create a bash script that changes the ssecuirity group named `rds-update-sg-rule`

```
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="rds-update-sg-rule"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

aws ec2 modify-security-group-rules \
    --group-id $DB_SG_ID \
    --security-group-rules "SecurityGroupRuleId=$DB_SG_RULE_ID,SecurityGroupRule={Description=GITPOD,IpProtocol=tcp,FromPort=5432,ToPort=5432,CidrIpv4=$GITPOD_IP/32}"
```

- Update the gitpod.yml so that it runs everytime a new workspace is launched. ADD a command step for postgres

```
     command: |
      export GITPOD_IP=$(curl ifconfig.me)
      source  "$THEIA_WORKSPACE_ROOT/backend-flask/rds-update-sg-rule"
```
 
- Update the `db-connect` file and run it using `prod`

![prod](https://user-images.githubusercontent.com/110903886/229313152-67cb0a9f-1eb0-43cf-b2a2-ecee9098db72.png)


- Load the schema into the database

![rds schema load](https://user-images.githubusercontent.com/110903886/229313278-f35a56c7-b201-4ab5-9a39-2d4be797c0f2.png)

## Setup Cognito post confirmation lambda

### Create the handler function
- Create lambda in same vpc as rds instance Python 3.8

![lambda1](https://user-images.githubusercontent.com/110903886/229589962-eb1e8316-56f7-4891-839d-48a5caeae1b1.png)
![lambda2](https://user-images.githubusercontent.com/110903886/229589957-3d6576f6-9688-4372-bfa7-ba64bab50c77.png)
![lambda3](https://user-images.githubusercontent.com/110903886/229589885-38887391-1ef8-434d-9c63-13a69d27f43a.png)


- Add a layer for psycopg2 with one of the below methods for development or production
![lambda layer](https://user-images.githubusercontent.com/110903886/229591544-bea02aed-fb41-4962-be71-3207067d1460.png)

The arn code provided didnt fit in for `us-east-1` so I got the right one from [here](https://github.com/jetbridge/psycopg2-lambda-layer)


### ENV variables needed for the lambda environment.

```
PG_HOSTNAME='cruddur-db-instance.czz1cuvepklc.ca-central-1.rds.amazonaws.com'
PG_DATABASE='cruddur'
PG_USERNAME='root'
PG_PASSWORD='huEE33z2Qvl383'
```
- Instead we'd use our $PROD_CONNECTION_URL as it already contains all those stated earlier

![set env](https://user-images.githubusercontent.com/110903886/229591612-ee408214-ebdc-4147-92ce-9fefc51d4e59.png)
![set env2](https://user-images.githubusercontent.com/110903886/229591605-04ef3ed6-437e-4e74-9bf0-ebdc4a355a83.png)


- Update The function code and deploy

```
import json
import psycopg2
import os

def lambda_handler(event, context):
    user = event['request']['userAttributes']
    print('userAttributes')
    print(user)

    user_display_name  = user['name']
    user_email         = user['email']
    user_handle        = user['preferred_username']
    user_cognito_id    = user['sub']
    try:
      print('entered-try')
      sql = f"""
         INSERT INTO public.users (
          display_name, 
          email,
          handle, 
          cognito_user_id
          ) 
        VALUES(%s,%s,%s,%s)
      """
      print('SQL Statement ----')
      print(sql)
      conn = psycopg2.connect(os.getenv('CONNECTION_URL'))
      cur = conn.cursor()
      params = [
        user_display_name,
        user_email,
        user_handle,
        user_cognito_id
      ]
      cur.execute(sql,*params)
      conn.commit() 

    except (Exception, psycopg2.DatabaseError) as error:
      print(error)
    finally:
      if conn is not None:
          cur.close()
          conn.close()
          print('Database connection closed.')
    return event
```
![lambda4](https://user-images.githubusercontent.com/110903886/229591694-4d1abe32-f635-4835-a9c7-62cf45eb16bf.png)

- Now lets add our trigger using cognito

![trigger](https://user-images.githubusercontent.com/110903886/229607366-d9284fe9-1794-4817-a929-8e2beaae3b20.png)
![trgger2](https://user-images.githubusercontent.com/110903886/229607354-ae407e4a-64cb-49c5-b532-60898cbf4210.png)

- Let's try signing up to see if the changes are working perfectly

![signup1](https://user-images.githubusercontent.com/110903886/229653306-e882abf3-c9f7-4874-9d08-4a2c0f5fc64f.png)

- To fix the issue, we'd update our vpc settings

![vpc config](https://user-images.githubusercontent.com/110903886/229653503-1fabc8d6-fbe8-42c5-9014-686aa9d38e4a.png)
![error](https://user-images.githubusercontent.com/110903886/229653518-15645dc7-8eb6-4b9f-bb8c-d1d601e136a9.png)

- To fix the above error, We have to edit the execution roles by creating a custom policy with the code below and attaching it.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeNetworkInterfaces",
        "ec2:CreateNetworkInterface",
        "ec2:DeleteNetworkInterface",
        "ec2:DescribeInstances",
        "ec2:AttachNetworkInterface"
      ],
      "Resource": "*"
    }
  ]
}
```

![role1](https://user-images.githubusercontent.com/110903886/229653427-f3b7e588-022d-4b5b-b79c-2b1369b71821.png)
![role2](https://user-images.githubusercontent.com/110903886/229653448-da04d9be-5ff1-4d1b-9a69-8e27b2719df2.png)
![role3](https://user-images.githubusercontent.com/110903886/229653787-6f1bf9f0-26ba-4672-8f2c-1148ce9dc09d.png)
![role4](https://user-images.githubusercontent.com/110903886/229653815-8c4bce68-9268-4f5a-9cdb-3766f0ef8b14.png)
![vpc added](https://user-images.githubusercontent.com/110903886/229653820-e7d17eb6-69a7-45bc-ba27-b45980958a05.png)

- I created a new account and my info wasn't added to the to the database. I got the below error from cloudwatch logs and fixed it

![logs](https://user-images.githubusercontent.com/110903886/229654120-a76b2d42-63af-4da8-8920-631571af4e2d.png)

![fix](https://user-images.githubusercontent.com/110903886/229654148-1497c2a5-f6e3-4614-b05f-722a3252d145.png)

- Created a new account and checked the database. Viola!!

![data](https://user-images.githubusercontent.com/110903886/229654267-6b086233-1bdd-4098-8209-0db415450ae8.png)

## Creating Activities

- Update your `create_activities.py`

```
from datetime import datetime, timedelta, timezone

from lib.db import db

class CreateActivity:
  def run(message, user_handle, ttl):
    model = {
      'errors': None,
      'data': None
    }

    now = datetime.now(timezone.utc).astimezone()

    if (ttl == '30-days'):
      ttl_offset = timedelta(days=30) 
    elif (ttl == '7-days'):
      ttl_offset = timedelta(days=7) 
    elif (ttl == '3-days'):
      ttl_offset = timedelta(days=3) 
    elif (ttl == '1-day'):
      ttl_offset = timedelta(days=1) 
    elif (ttl == '12-hours'):
      ttl_offset = timedelta(hours=12) 
    elif (ttl == '3-hours'):
      ttl_offset = timedelta(hours=3) 
    elif (ttl == '1-hour'):
      ttl_offset = timedelta(hours=1) 
    else:
      model['errors'] = ['ttl_blank']

    if user_handle == None or len(user_handle) < 1:
      model['errors'] = ['user_handle_blank']

    if message == None or len(message) < 1:
      model['errors'] = ['message_blank'] 
    elif len(message) > 280:
      model['errors'] = ['message_exceed_max_chars'] 

    if model['errors']:
      model['data'] = {
        'handle':  user_handle,
        'message': message
      }   
    else:
      expires_at = (now + ttl_offset)
      uuid = CreateActivity.create_activity(user_handle,message,expires_at)

      object_json = CreateActivity.query_object_activity(uuid)
      model['data'] = object_json
    return model

  def create_activity(handle, message, expires_at):
    sql = db.template('activities','create')
    uuid = db.query_commit(sql,{
      'handle': handle,
      'message': message,
      'expires_at': expires_at
    })
    return uuid
  def query_object_activity(uuid):
    sql = db.template('activities','object')
    return db.query_object_json(sql,{
      'uuid': uuid
    })
```

- `home_activities.py`

```
from datetime import datetime, timedelta, timezone
from opentelemetry import trace

from lib.db import db

#tracer = trace.get_tracer("home.activities")

class HomeActivities:
  def run(cognito_user_id=None):
    #logger.info("HomeActivities")
    #with tracer.start_as_current_span("home-activites-mock-data"):
    #  span = trace.get_current_span()
    #  now = datetime.now(timezone.utc).astimezone()
    #  span.set_attribute("app.now", now.isoformat())
    sql = db.template('activities','home')
    results = db.query_array_json(sql)
    return results    
```

- `db.py`

```
from psycopg_pool import ConnectionPool
import os
import re
import sys
from flask import current_app as app

class Db:
  def __init__(self):
    self.init_pool()

  def template(self,*args):
    pathing = list((app.root_path,'db','sql',) + args)
    pathing[-1] = pathing[-1] + ".sql"

    template_path = os.path.join(*pathing)

    green = '\033[92m'
    no_color = '\033[0m'
    print("\n")
    print(f'{green} Load SQL Template: {template_path} {no_color}')

    with open(template_path, 'r') as f:
      template_content = f.read()
    return template_content

  def init_pool(self):
    connection_url = os.getenv("CONNECTION_URL")
    self.pool = ConnectionPool(connection_url)
  # we want to commit data such as an insert
  # be sure to check for RETURNING in all uppercases
  def print_params(self,params):
    blue = '\033[94m'
    no_color = '\033[0m'
    print(f'{blue} SQL Params:{no_color}')
    for key, value in params.items():
      print(key, ":", value)

  def print_sql(self,title,sql):
    cyan = '\033[96m'
    no_color = '\033[0m'
    print(f'{cyan} SQL STATEMENT-[{title}]------{no_color}')
    print(sql)
  def query_commit(self,sql,params={}):
    self.print_sql('commit with returning',sql)

    pattern = r"\bRETURNING\b"
    is_returning_id = re.search(pattern, sql)

    try:
      with self.pool.connection() as conn:
        cur =  conn.cursor()
        cur.execute(sql,params)
        if is_returning_id:
          returning_id = cur.fetchone()[0]
        conn.commit() 
        if is_returning_id:
          return returning_id
    except Exception as err:
      self.print_sql_err(err)
  # when we want to return a json object
  def query_array_json(self,sql,params={}):
    self.print_sql('array',sql)

    wrapped_sql = self.query_wrap_array(sql)
    with self.pool.connection() as conn:
      with conn.cursor() as cur:
        cur.execute(wrapped_sql,params)
        json = cur.fetchone()
        return json[0]
  # When we want to return an array of json objects
  def query_object_json(self,sql,params={}):

    self.print_sql('json',sql)
    self.print_params(params)
    wrapped_sql = self.query_wrap_object(sql)

    with self.pool.connection() as conn:
      with conn.cursor() as cur:
        cur.execute(wrapped_sql,params)
        json = cur.fetchone()
        if json == None:
          "{}"
        else:
          return json[0]
  def query_wrap_object(self,template):
    sql = f"""
    (SELECT COALESCE(row_to_json(object_row),'{{}}'::json) FROM (
    {template}
    ) object_row);
    """
    return sql
  def query_wrap_array(self,template):
    sql = f"""
    (SELECT COALESCE(array_to_json(array_agg(row_to_json(array_row))),'[]'::json) FROM (
    {template}
    ) array_row);
    """
    return sql
  def print_sql_err(self,err):
    # get details about the exception
    err_type, err_obj, traceback = sys.exc_info()

    # get the line number when exception occured
    line_num = traceback.tb_lineno

    # print the connect() error
    print ("\npsycopg ERROR:", err, "on line number:", line_num)
    print ("psycopg traceback:", traceback, "-- type:", err_type)

    # print the pgcode and pgerror exceptions
    print ("pgerror:", err.pgerror)
    print ("pgcode:", err.pgcode, "\n")

db = Db()
```



- Create a dir called `sql` and a sub dir called `activities`. Create 3 `.sql` files named `create.sql`, `home.sql` and `object.sql` inside the folder. Update them with the codes below

`create.sql`

```
INSERT INTO public.activities (
  user_uuid,
  message,
  expires_at
)
VALUES (
  (SELECT uuid 
    FROM public.users 
    WHERE users.handle = %(handle)s
    LIMIT 1
  ),
  %(message)s,
  %(expires_at)s
) RETURNING uuid;
```

`home.sql`

```
SELECT
  activities.uuid,
  users.display_name,
  users.handle,
  activities.message,
  activities.replies_count,
  activities.reposts_count,
  activities.likes_count,
  activities.reply_to_activity_uuid,
  activities.expires_at,
  activities.created_at
FROM public.activities
LEFT JOIN public.users ON users.uuid = activities.user_uuid
ORDER BY activities.created_at DESC
```

`object.sql`

```
SELECT
  activities.uuid,
  users.display_name,
  users.handle,
  activities.message,
  activities.created_at,
  activities.expires_at
FROM public.activities
INNER JOIN public.users ON users.uuid = activities.user_uuid 
WHERE 
  activities.uuid = %(uuid)s
```

- Tested the crud button but got an error

![jjjj](https://user-images.githubusercontent.com/110903886/230655234-38c47430-5856-46e9-8d38-a024881aa20d.png)

** FIX **

- Add the activityform component in `pages/HomeFeedPage.js` to pass the user_handle prop.
![user1](https://user-images.githubusercontent.com/110903886/230655533-f00a00a0-0e28-4851-8b0e-6e8ab78acb7c.png)

- Update fetch request body section of `components/ActivityForm.js`, to include the user_handle.
![user2](https://user-images.githubusercontent.com/110903886/230655603-95d9aa9c-f249-4b34-ade1-cd2bdf1f3840.png)

- Assign the `user_handle` variable under `api/activities/route` in `app.py`

**The changes would ensure that the user_handle prop is passed correctly and would get included in the fetch request. So that the server can retrieve it from the request payload.**

![user3](https://user-images.githubusercontent.com/110903886/230656095-677906df-ffb2-4b59-a582-a090a63f58b5.png)

- Tested the `crud` again

![user4](https://user-images.githubusercontent.com/110903886/230656152-839c96b8-0436-4f91-9524-3f0a545715d6.png)


![check1](https://user-images.githubusercontent.com/110903886/230656192-d22ccf64-a253-4cec-8888-291e6be41fd6.png)





