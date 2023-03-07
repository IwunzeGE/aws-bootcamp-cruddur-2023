# Week 3 — Decentralized Authentication

## Cognito
- Log in as IAM User on the console
- Configure the AWS CLI credentials
- From the console launch 'Cognito' and create a user pool

The user pool is created in 6 steps - We'd want authentications with username (not case sensitive) and emails

![cognito1](https://user-images.githubusercontent.com/110903886/223525108-fc6ad95f-45ca-4e57-91ab-535bb72176a7.png)

Step 2&3 focuses more on secuirity requirements for sign-in ans sign-up - password requirements, MFA setup, account recovery options etc.

![cognito2](https://user-images.githubusercontent.com/110903886/223527078-ffa878db-f8eb-4102-9e10-fa07274821ce.png)

Step 4 configures how you'd want your messages to be sent. If you are using SES, it will require you set up a verified SES profile. So we used Cognito defaults although it is limited to just 50 emails per day

Step 5 works around integrating your application 

Review and create the pool at Step 6

![cognito3](https://user-images.githubusercontent.com/110903886/223534566-d0a2107b-6554-4f74-9532-1efd6e7c5a4f.png)
