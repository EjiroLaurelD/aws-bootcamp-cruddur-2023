# Week 3 — Decentralized Authentication
- I created a user pool for my app on AWS cognito
![userpool](./assets/userpool.png)

- I installed aws amplify
`npm i aws-amplify --save`

- I set my cognito user pool user with this code on the command line
`aws cognito-idp admin-set-user-password --username laurel --password mypassword --user-pool-id us-east-1_cqUcwvnM9 --permanent`

- I implemented recovery password
![recovery](./assets/recovery.png)
