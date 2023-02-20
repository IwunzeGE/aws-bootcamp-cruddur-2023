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

f. Set the budget amount and timeframe (e.g. monthly budget of $10 for the next 6 months)

g. Choose the resources you want to include in the budget (e.g. all resources in your AWS account, specific AWS services, specific tags, etc.)

h. Set up alerts to be notified when your usage or costs exceed the budget amount.

### Creating IAM USERS
- Log in to your AWS account using your credentials.
- Navigate to the IAM console by selecting "Services" from the top menu, then selecting "IAM" under the "Security, Identity & Compliance" category.
- In the left navigation pane, click "Users".
- Click the "Add user" button.
- Enter a name for the new user and select the access type that you want to grant to the user. You can select either "Programmatic access" for API access or "AWS Management Console access" for console access, or both.
- Set a password for the user, or choose the option to allow the user to create their own password.
- Configure the user's permissions by selecting the appropriate checkboxes in the "Set permissions" section. You can either assign the user to an existing group with predefined permissions or create a custom policy for the user.
- Review your settings and click the "Create user" button.
- Once the user is created, you will see their credentials, including their access key and secret access key, which are required for programmatic access. You can also download a CSV file containing the user's login information and credentials for future reference.

![IAM3](https://user-images.githubusercontent.com/110903886/220133151-615a91d2-2e3b-4d5d-9c8b-2d15efded12e.png)
![IAM1](https://user-images.githubusercontent.com/110903886/220133158-977f247a-8153-4e5d-a444-e153f2d6ea1f.png)
![IAM2](https://user-images.githubusercontent.com/110903886/220133163-daf3cc5e-1c3f-49bf-a9d8-53d03fa6e091.png)

###Setting Up the AWS CLI on Gitpod

- Install the CLI via the terminal using this code

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

- Configure your credentials using `aws configure`

You will be prompted to enter your AWS Access Key ID, AWS Secret Access Key, default region name, and default output format.

Enter your Access Key ID and Secret Access Key, which can be found in your AWS Management Console under the "IAM" service.

Enter the default region name and output format. The region is the geographical location of your AWS resources. You can find a list of available regions in the AWS documentation. The output format can be either "json", "text", or "table".

```
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

- Once you have entered all the required information, the AWS CLI is now configured and ready to use. 

### Setting up Budgets via the CLI

1. The budget will be created using this command but first you'll have to set up the budget.json and notification-with-subscribers.json file

```aws budgets create-budget \
    --account-id 111122223333 \
    --budget file://budget.json \
    --notifications-with-subscribers file://notifications-with-subscribers.json
```
- Create two files named budget.json and notification-with-subscribers.json and edit them with the below code respectively

```
{
    "BudgetLimit": {
        "Amount": "100",
        "Unit": "USD"
    },
    "BudgetName": "Example Tag Budget",
    "BudgetType": "COST",
    "CostFilters": {
        "TagKeyValue": [
            "user:Key$value1",
            "user:Key$value2"
        ]
    },
    "CostTypes": {
        "IncludeCredit": true,
        "IncludeDiscount": true,
        "IncludeOtherSubscription": true,
        "IncludeRecurring": true,
        "IncludeRefund": true,
        "IncludeSubscription": true,
        "IncludeSupport": true,
        "IncludeTax": true,
        "IncludeUpfront": true,
        "UseBlended": false
    },
    "TimePeriod": {
        "Start": 1477958399,
        "End": 3706473600
    },
    "TimeUnit": "MONTHLY"
}
```

```
[
    {
        "Notification": {
            "ComparisonOperator": "GREATER_THAN",
            "NotificationType": "ACTUAL",
            "Threshold": 80,
            "ThresholdType": "PERCENTAGE"
        },
        "Subscribers": [
            {
                "Address": "example@example.com",
                "SubscriptionType": "EMAIL"
            }
        ]
    }
]
```

Press Enter to run the command. If successful, you should see the details of the newly created budget.


## Conceptual Diagram

![Screenshot (53)](https://user-images.githubusercontent.com/110903886/219217558-31c93fae-8f29-4303-9579-f64e03f6f80b.png)

[Link to the conceptual image](https://lucid.app/lucidchart/d8ada944-82ba-4131-8eac-46233bdfbae8/edit?viewport_loc=-774%2C-96%2C3330%2C1461%2C0_0&invitationId=inv_4141424b-aae1-4e6d-90a2-a8c78987c1e7)


## Logical Diagram

![Screenshot (54)](https://user-images.githubusercontent.com/110903886/220132525-461f7839-5f6c-4068-82c3-6bbbeb89bc96.png)

[Link to the logical image](https://lucid.app/lucidchart/891abc20-19d0-4c08-ba22-22b47ebf0cb3/edit?viewport_loc=-586%2C-1278%2C3042%2C1364%2C0_0&invitationId=inv_be692115-abdc-4798-a5f1-0222a559edd6)
