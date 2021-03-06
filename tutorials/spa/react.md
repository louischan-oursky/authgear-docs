---
description: >-
  Follow this quickstart tutorial to add authentication to your React
  application
---

# React

## Setup Application in Authgear

Signup for an account in [https://portal.authgearapps.com/](https://portal.authgearapps.com/) and create a Project.

![Continue to Portal to create a new Application in the Project](../../.gitbook/assets/continue-to-portal.png)

After that, we will need to create an Application in the Project Portal.

### Create an application in the Portal

1. Go to **Applications** in your project portal.
2. Click **Add Application** in the top right corner.
3. Input the name of your application. This is for reference only. 
4. Decide a path in your website that users will be redirected to after they have authenticated with Authgear. Add the URI to **Redirect URIs**. e.g.`http://localhost:3000/auth-redirect`  for this tutorial.
5. Select the **Issue JWT as access token** option. We will use Token-based authentication for this tutorial. 
6. Click "Save" and keep the **Client ID**. You can also obtain it from the Applications list later.

### Add your website to allowed origins

1. Go to **Applications** &gt; **Allowed Origins**.
2. Add your website origin to the allowed origins. Set to`localhost:3000` for this tutorial.
3. Click "Save".

## Create a sample application

If you don't already have an existing React application, you can create one using **Create React App**. Skip this part if you are integrating Authgear to your existing application.

```bash
# Create the application using Create React App
npx create-react-app my-app
# Move into the project directory
cd my-app
# Add the react router as we will be using it later
npm install --save react-router-dom
```

## Install the Authgear Web SDK

Run the following command within your React project directory to install the Authgear Web SDK

```bash
npm install --save --save-exact @authgear/web
```

## Create an Authentication Provider

## Add Login to Your Application

## Add Logout to Your Application

## Protecting Routes

## Calling an API

To access restricted resources on your application server, the HTTP requests should include the access token in their Authorization headers. The Web SDK provides a `fetch` function which automatically handle this, or you can get the token with `authgear.accessToken`.

#### Option 1: Using fetch function provided by Authgear SDK

Authgear SDK provides the `fetch` function for you to call your application server. This `fetch` function will include the Authorization header in your application request, and handle refresh access token automatically. The `authgear.fetch` implements [fetch](https://fetch.spec.whatwg.org/).

```javascript
authgear
    .fetch("YOUR_SERVER_URL")
    .then(response => response.json())
    .then(data => console.log(data));
```

#### Option 2: Add the access token to the HTTP request header

You can get the access token through `authgear.accessToken`. Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make the network call only if the access token has expired. Include the access token into the Authorization header of the application request.

```javascript
authgear
    .refreshAccessTokenIfNeeded()
    .then(() => {
        // access token is ready to use
        // accessToken can be string or undefined
        // it will be empty if user is not logged in or session is invalid
        const accessToken = authgear.accessToken;

        // include Authorization header in your application request
        const headers = {
            Authorization: `Bearer ${accessToken}`
        };
    });
```

{% hint style="warning" %}
This page is working in progress, please see [Get Started: Web SDK ](../../get-started/website.md)for general guides of using the Authgear Web SDK
{% endhint %}

