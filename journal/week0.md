# Week 0 — Billing and Architecture

## Requirements
1. AWS account
2. Lucid account

## AWS Budgets
AWS budgets allow you to set custom cost and usage targets for your resources, and receive alerts when your usage exceeds those targets. I was able to set up budgets via the console or AWS CLI

### Setting up Budgets via the Console

a. Logged in to your AWS console

b. Click on the "Billing & Cost Management" option


c. Select "Budgets" from the dropdown menu

d. Click on the "Create budget" button

e. Select the budget type you want to create (e.g. Cost budget, Usage budget, etc.)

- Cost budgets: You can set a budget for your AWS costs, and receive alerts when your costs exceed your budget.
- Usage budgets: You can set a budget for your AWS usage, and receive alerts when your usage exceeds your budget.
- Cost and usage budgets: You can set a budget for both your AWS costs and usage, and receive alerts when either exceeds the budget.

f. Set the budget amount and timeframe (e.g. monthly budget of $500 for the next 6 months)

g. Choose the resources you want to include in the budget (e.g. all resources in your AWS account, specific AWS services, specific tags, etc.)

h. Set up alerts to be notified when your usage or costs exceed the budget amount.
### Creating IAM USERS

![IAM3](https://user-images.githubusercontent.com/110903886/220133151-615a91d2-2e3b-4d5d-9c8b-2d15efded12e.png)
![IAM1](https://user-images.githubusercontent.com/110903886/220133158-977f247a-8153-4e5d-a444-e153f2d6ea1f.png)
![IAM2](https://user-images.githubusercontent.com/110903886/220133163-daf3cc5e-1c3f-49bf-a9d8-53d03fa6e091.png)

### Setting up Budgets via the CLI

a. Install the AWS CLI on your local machine if you haven't already. You can download and install it from the official AWS website.

b. Create a budget in the AWS Management Console, noting the budget name and the email addresses of the recipients.

c. In your terminal, enter the following command to create a new budget:

```
aws budgets create-budget --account-id your-account-id --budget budget-name --budget-type COST --limit-amount limit-amount --limit-unit limit-unit --time-unit MONTHLY --start-date start-date --end-date end-date --email-subject email-subject --subscriber-email-addresses email-address-1 email-address-2

```

*Replace the your-account-id with your AWS account ID, budget-name with the name of your budget, limit-amount with the budget amount, and limit-unit with the currency code (e.g., USD). Replace start-date and end-date with the start and end dates of the budget period (in YYYY-MM-DD format). You can also add multiple email addresses by separating them with spaces after the --subscriber-email-addresses flag.*

d. Press Enter to run the command. If successful, you should see the details of the newly created budget.

## Conceptual Diagram

![Screenshot (53)](https://user-images.githubusercontent.com/110903886/219217558-31c93fae-8f29-4303-9579-f64e03f6f80b.png)

[Link to the conceptual image](https://lucid.app/lucidchart/d8ada944-82ba-4131-8eac-46233bdfbae8/edit?viewport_loc=-774%2C-96%2C3330%2C1461%2C0_0&invitationId=inv_4141424b-aae1-4e6d-90a2-a8c78987c1e7)


## Logical Diagram

![Screenshot (54)](https://user-images.githubusercontent.com/110903886/220132525-461f7839-5f6c-4068-82c3-6bbbeb89bc96.png)

[Link to the logical image](https://lucid.app/lucidchart/891abc20-19d0-4c08-ba22-22b47ebf0cb3/edit?viewport_loc=-586%2C-1278%2C3042%2C1364%2C0_0&invitationId=inv_be692115-abdc-4798-a5f1-0222a559edd6)
