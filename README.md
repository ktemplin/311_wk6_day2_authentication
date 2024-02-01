# Express Authentication

PLEASE NOTE THAT PRIOR TO THIS ASSIGNMENT, YOU NEED TO HAVE FINISHED THE SECTION TITLED "Authentication with Auth0" IN THE PRE-HOMEWORK

### ‼️ ⚠️  ***ATTENTION*** ⚠️ ‼️

There is a typo in this repo that will cause it to crash! Please go to to `/controllers/auth.js` line 1 and remove the `1` character from the beginning of the line, as seen [here](https://github.com/AustinCodingAcademy/311_wk6_day2_authentication/blob/master/controllers/auth.js#L1).

There are some known issues with dependencies for this project on Windows. Before doing any of the following setup, please do the following: 

1. Run `npm audit fix --force` (this will fix dependency issues on Windows)
2. Run `npm install`
3. Run `npm start`

If you're still seeing errors related to `jwt` not being a function, go to `middleware/index.js` and change the import on line 2 for `jwt` to this: 

```js
const { expressjwt: jwt } = require("express-jwt");
```

Now you may proceed with the below setup steps 👍

## Setup

1. Initialize and run the app: `npm install` to watch the dependencies be downloaded

1. Create a `.env` file with the following structure. Alter the fields below to match your own values, using the Auth0 account you created. There are screenshots below to help you find these values in your Auth0 dashboard.

```yaml
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_local_mysql_password_here
DB_DEFAULT_SCHEMA=admin
AUTH0_IDENTITY=express-auth-app // or whatever you called the project in Auth0
AUTH0_DOMAIN=dev-randomstring.us.auth0.com
AUTH0_CLIENT_ID=asdfdsfsafda
AUTH0_CLIENT_SECRET=asdfsadfdsfasdf
```
> *NOTE: Don't include quotes in the `.env` file.


You can find the values for the `AUTH0` environment variables by navigating to the places in your Auth0 dashboard, as shown in the images below.

<img width="1270" alt="Screen Shot 2023-02-08 at 1 51 31 PM" src="https://user-images.githubusercontent.com/20386502/217671550-84a20286-cdf9-4b85-9d2d-aefa18b6190d.png">

Be sure to add the domain exactly as it appears in your dashboard, without `http(s)://`:

![Screen Shot 2023-02-08 at 3 34 12 PM](https://user-images.githubusercontent.com/20386502/217676224-67f22d61-68cf-47a4-bafe-759557e0ab52.png)


The app is using `nodemon` so you can use `npm start` and any changes made (and saved) will cause the server to restart automatically.

If you get errors after adding your database credentials to the `.env` file in the instructions below, follow these commands to upgrade to `mysql2`:
  - `npm uninstall mysql`
  - `npm install mysql2`
  - Modify all places in the app that require `mysql`: 
    - Change `const mysql = require('mysql')` to `const mysql = require('mysql2')`

Finally, in MySQL Workbench, run the `initialize.sql` script that is included in this project. You will run this on the "admin" database. You can simply copy the sql from the file into your MySQL Workbench console. Follow the steps under the `imgs` folder if you are having trouble with this.

## Overview

Note: This is a tough project, but hang in there and try to understand as much as possible. Auth0 is a popular framework for authentication. A lot of the setup is done for us.

The `routes/controllers`, SQL statements and basic setup has been done for us. Our job is now to complete the functions in the middleware folder and then use them in our routes.

## GET users

In Postman or a web browser, navigate to `http://localhost:4001/users/` (a GET request) and you should be able to see a list of all the users in your database. This is information that we will leave public.

## POST / PUT / DELETE

These routes are for manipulating the data and these are things that we ideally want someone to be logged in for before they are able to work with the data. To start, we will create a middleware function (that Auth0 provides us).

### Middleware

In the `middleware/index.js` file, locate the function called `checkJwt`. We need to apply this middleware to the routes you want to protect. Before you do that though... go to Postman and send a POST request to `http://localhost:4001/users/` with no body. This should add a pre-selected user to your DB. You should have gotten a response that looks like this:

```json
{
    "newId": 501
}
```

In order to prevent this, we need to go to that route, the third one down in the `routers/users.js` file, and add `checkJwt` in between the path and the request/response function. The final result should look like this:

```js
// routes/users line 10
router.post('/', checkJwt, usersController.createUser)
```

Now go ahead and attempt to make that POST request again in Postman. The one to `http://localhost:4001/users/`. Try it a couple of times. You should now get a response with a 401 status code and a body that has `UnauthorizedError: No authorization token was found` in it. That's good news, we are now blocking requests to this endpoint until people are authenticated. Add that same middleware to the rest of the routes (that are not GET requests) in the `routers/users.js` file.

## Authenticating

So how do users authenticate? We've seen how we can block them but now we actually need certain users to have access. So we need to send the bearer token (jwt). First we'll do a test run.

### Testing the JWT

From the Auth0 dashboard in your browser, navigate to APIs (on the left side) -> My Express App (the API you created in pre-homework) -> Test. Midway down the page in the code block labeled "Response", copy the "access token" to your clipboard. It should start with `eyJ0eXAiOiJKV1QiLCJh...` or some close combination of characters.

Now in Postman, we're going to execute that same POST request but this time we are going to add a header. A header as we know is a piece of information that gets sent along with the request. In Postman go to the "headers" column and give it the name "Authorization" and the value of your token preceded by the word "Bearer". So it will look something like:

`Authorization: Bearer eyJ0eXAiOiJKV1QiLCJh...`

Execute the request and notice that you are allowed to add users again and see a response that looks like this:

```json
{
    "newId": 502
}
```

### Obtaining the JWT

Ok so we now have protected routes and some users can access them if they have the appropriate token but where do they get that token from? We need to create a workflow that sends back a token when a user logs in. We need to do that by calling an Auth0 endpoint during the login endpoint.

Find the "login" function in [controllers/auth.js](./controllers/auth.js). You'll see that the call to the Auth0 endpoint is mostly complete but we still need to do a few things.

1. Set the default directory on your Auth0 account to "Username-Password-Authentication". You can do this by clicking "Settings" in the bottom of the left sidebar. On the "General" tab, scroll down to the "API Authorization Settings" box and add `Username-Password-Authentication` to the "Default Directory" input, as shown in the screenshot below: 

![Screen Shot 2023-02-08 at 5 12 22 PM](https://user-images.githubusercontent.com/20386502/217694674-526453d3-c28e-4c1a-be08-27d1073e2e14.png)

Next, go to `Applications` --> [click the project you just created] --> Scroll all the way down to "Advanced Settings." Make sure that the option "Password" is checked: 

<img width="1282" alt="Screen Shot 2023-02-10 at 3 16 44 PM" src="https://user-images.githubusercontent.com/20386502/218220646-04457bf5-438f-4eb0-8550-b05b52498cf4.png">


2. Now we need to create a couple of users for our application to use. We will do this manually for now. Go to the main page of your Auth0 dashboard in your browser, select "Users & Roles" -> "User", then click "Create User". Leave the default connection type and enter an email and password for your user. For example:

```yaml
email: test@example.com
password: Password!
```

Remember this information because we are about to use it again. This time go to Postman and submit a request to the `/auth/login` route that's already been created for us in this application. The full request will be a POST to `http://localhost:4001/auth/login` with the body:

```json
{
    "username": "test@example.com",
    "password": "Password!"
}
```

If everything worked correctly you should have received an "access_token" in return. That's the token that we can use to send to your endpoints after a user logs in. Keep in mind that the "users" are now stored in Auth0 and are different from the users we have in our database. Those database users are just dummy data at this point. We won't need to actually store information about our users because Auth0 will do it for us.

If you are having issues you may refer to the last screenshot above and make sure that you have `Username-Password-Authentication` in the "Default Directory" input.

## BONUS - logger

Create a function called `logger` in the `middleware/index.js` file. Its purpose will be to log the route and date/time that each request happened. The outline of the function will look like this:

```js
const logger = (req, res, next) => {

}
```

Inside of this function we will put a `console.log` statement with three arguments separated by a comma:

1. The string, `'Logging route:'`
2. The request path ex. `/users`
3. The date/time in ISO format. Ex. `new Date().toISOString()`

Remember to call the `next()` function in order to continue. Otherwise, the API call will get hung up in this middleware function.

Import this logger function into the main `index.js` file: `const { logger } = require('./middleware')`

Between the `bodyParser` and the users router add the following: `app.use(logger)`

This is an example of application specific middleware. Every route will now pass through our logger function and log the path and the date/time that the request was made. This would be useful for determining our most popular routes.

## Summary

If all went according to plan we now have an API that is locked down with authentication and we have also added middleware on all of our routes that logs the current request and the associated date/time.

