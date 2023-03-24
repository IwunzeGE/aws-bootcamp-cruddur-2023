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
 

Now, lets provisioning an RDS instance with db named 'cruddur' from the CLI using the code below.

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
  
  
