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
- Check if it works by signing in with a random credential. You should get this

![check0](https://user-images.githubusercontent.com/110903886/223864963-efdca9a2-1955-4e27-8e00-5ee7ea7fe185.png)

- We'd want the error message to display in our webapp not just the inspect console. So tweak the code in `Signin.js`

Replace that part of the code with this

```
const onsubmit = async (event) => {
    setErrors('')
    console.log()
    event.preventDefault();
      Auth.signIn(email, password)
        .then(user => {
          localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
          window.location.href = "/"
        })
        .catch(error => { 
      if (error.code == 'UserNotConfirmedException') {
        window.location.href = "/confirm"
      }
      setErrors(error.message)
    });
    return false
  }
```

- Now lets check,

![fixed signin error](https://user-images.githubusercontent.com/110903886/223865623-8219b412-d7c4-44ca-a3b1-5ecc57c19fb2.png)

- Create a user in the cognito user pool form the console and try using the credentials to login

![create user](https://user-images.githubusercontent.com/110903886/223865830-9a00a62c-6b78-4b3e-bc5a-b3db3723932e.png)

I received an email alert upon creating the user
![email alert](https://user-images.githubusercontent.com/110903886/223865850-1da81fb4-7f3e-4638-b4f0-187c4db18edb.png)

- If login fails and gives the below error, Then we might need to force the password reset via the CLI. This is because user confirmation wasn't allowed from the console.

![user login cannot read properties error](https://user-images.githubusercontent.com/110903886/223866654-7884243d-9a9b-4aa7-badf-c851a3e8f829.png)

From the CLI,

`aws cognito-idp admin-set-user-password --username iwunzege --password Password!! --user-pool-id us-east-1_wy78euiKi --permanent`

- Upon sign in, it was successful
![Login success but no attributes](https://user-images.githubusercontent.com/110903886/223866122-d35b2704-49fe-4091-a2bc-027df03a88b0.png)

- From the AWS console, edit the `User Atrributes`

![user attributes](https://user-images.githubusercontent.com/110903886/223868008-61696747-56fe-45ae-be73-8b09c970efe5.png)

- Refresh the frontend page to see if it worked

![Login success with attributes](https://user-images.githubusercontent.com/110903886/223869518-d8d5d4d8-7e83-4a59-8425-fea2f1bd747b.png)






- Update the Signup Page

```
import { Auth } from 'aws-amplify';


const [cognitoErrors, setCognitoErrors] = React.useState('');


const onsubmit = async (event) => {
  event.preventDefault();
  setCognitoErrors('')
  try {
      const { user } = await Auth.signUp({
        username: email,
        password: password,
        attributes: {
            name: name,
            email: email,
            preferred_username: username,
        },
        autoSignIn: { // optional - enables auto sign in after user is confirmed
            enabled: true,
        }
      });
      console.log(user);
      window.location.href = `/confirm?email=${email}`
  } catch (error) {
      console.log(error);
      setCognitoErrors(error.message)
  }
  return false
}


let errors;
if (cognitoErrors){
  errors = <div className='errors'>{cognitoErrors}</div>;
}

//before submit component
{errors}
```

## Configuring the Backend

- Add `Flask-AWSCognito` to the `requirements.txt` in the backend and install it using pip.

- Change the CORS part of the code in the `app.py` file
```
cors = CORS(
  app, 
  resources={r"/api/*": {"origins": origins}},
  headers=['Content-Type', 'Authorization'], 
  expose_headers='Authorization',
  methods="OPTIONS,GET,HEAD,POST"
)
```

