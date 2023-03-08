# Week 3 — Decentralized Authentication

## Set-up Cognito
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

## Install AWS Amplify
`npm i aws-amplify --save`


## Configure Amplify
- Lets hook up our cognito pool to our code in the `App.js`

```
import { Amplify } from 'aws-amplify';


Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_AWS_PROJECT_REGION,
  //"aws_cognito_identity_pool_id": process.env.REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_AWS_USER_POOLS_WEB_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});
```

- Set the env vars

```
REACT_AWS_PROJECT_REGION: "${AWS_DEFAULT_REGION}"
# REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID: " "
REACT_APP_AWS_COGNITO_REGION: "{AWS_DEFAULT_REGION}"
REACT_APP_AWS_USER_POOLS_ID: "us-east-1_TEsk63AWn"
REACT_APP_CLIENT_ID: "gd4723061qik6p1sp8j7jeor2"
```

## Conditionally show components based on logged in or logged out

- Inside our `HomeFeedPage.js`

```
import { Auth } from 'aws-amplify';

// set a state
const [user, setUser] = React.useState(null);

// check if we are authenicated
const checkAuth = async () => {
  Auth.currentAuthenticatedUser({
    // Optional, By default is false. 
    // If set to true, this call will send a 
    // request to Cognito to get the latest user data
    bypassCache: false 
  })
  .then((user) => {
    console.log('user',user);
    return Auth.currentAuthenticatedUser()
  }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
  })
  .catch((err) => console.log(err));
};

// check when the page loads if we are authenicated
React.useEffect(()=>{
  loadData();
  checkAuth();
}, [])
```

- We'll update ProfileInfo.js

```
import { Auth } from 'aws-amplify';

const signOut = async () => {
  try {
      await Auth.signOut({ global: true });
      window.location.href = "/"
  } catch (error) {
      console.log('error signing out: ', error);
  }
}
```

- Update the Signin Page

```
import { Auth } from 'aws-amplify';

const [cognitoErrors, setCognitoErrors] = React.useState('');

const onsubmit = async (event) => {
  setCognitoErrors('')
  event.preventDefault();
  try {
    Auth.signIn(username, password)
      .then(user => {
        localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
        window.location.href = "/"
      })
      .catch(err => { console.log('Error!', err) });
  } catch (error) {
    if (error.code == 'UserNotConfirmedException') {
      window.location.href = "/confirm"
    }
    setCognitoErrors(error.message)
  }
  return false
}

let errors;
if (cognitoErrors){
  errors = <div className='errors'>{cognitoErrors}</div>;
}

// just before submit component
{errors}
```

