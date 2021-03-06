---
description: >-
  How to make authorized request to your application server after login with
  Authgear
---

# Using SDK to call your application server

## Using Authgear SDK to call your application server

In this section, we are going to explain how to make an authorized request to your application server by using Authgear SDK. If you are using session cookies in your web application, you can skip this section as session cookies are handled by the browser automatically.

## Setup Overview

1. To determine which user is calling your server, you will need to include the Authorization header in every request that send to your application server.
2. You will need to set up [Backend integration](../getting-started/backend-integration/nginx.md), Authgear will help you to resolve the Authorization header to determine whether the incoming HTTP request is authenticated or not.

In the below section, we will explain how to set up SDK to include Authorization header to your application requests.

## SDK setup

* Configure the Authgear SDK with the Authgear endpoint and client id. The SDK must be properly configured before use, you can call `configure` multiple times but you should only need to call it once. No network call will be triggered during `configure`.

{% tabs %}
{% tab title="Javascript" %}
```javascript
authgear
    .configure({
        clientID: "a_random_generated_string",
        endpoint: "http://localhost:3000",
    })
    .then(() => {
        // configured successfully
    })
    .catch((e) => {
        // failed to configured
    });
```
{% endtab %}

{% tab title="iOS" %}
```swift
let authgear = Authgear(clientId: clientId, endpoint: endpoint)
authgear.configure() { result in
    switch result {
    case .success():
        // configured successfully
    case let .failure(error):
        // failed to configured
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
ConfigureOptions configureOptions = new ConfigureOptions();
Authgear authgear = new Authgear(getApplication(), clientID, endpoint, null);
authgear.configure(configureOptions, new OnConfigureListener() {
    @Override
    public void onConfigured() {
        // configured successfully
    }

    @Override
    public void onConfigurationFailed(@NonNull Throwable throwable) {
        // failed to configured
    }
});
```
{% endtab %}
{% endtabs %}

* `sessionState` reflect the user logged in state. Right after configure, the session state only reflects the SDK local state. That means even `sessionState` is `AUTHENTICATED`, the session may be invalid if it is revoked remotely. You will only know that after calling the server, call `fetchUserInfo` as soon as it is proper to do so, e.g. when the device goes online.

{% tabs %}
{% tab title="Javascript" %}
```javascript
// value can be NO_SESSION or AUTHENTICATED
// After authgear.configure, it only reflect SDK local state.
let sessionState = authgear.sessionState;

if (sessionState === "AUTHENTICATED") {
    authgear
        .fetchUserInfo()
        .then((userInfo) => {
            // sessionState is now up to date
            // read the userInfo if needed
        })
        .catch((e) => {
            // sessionState is now up to date
            // it will change to NO_SESSION if the session is invalid
        });
}
```
{% endtab %}

{% tab title="iOS" %}
```swift
// value can be .noSession or .authenticated.
// After authgear.configure, it only reflect SDK local state.
let sessionState = authgear.sessionState

// call fetchUserInfo to see if the session is valid
if authgear.sessionState == .authenticated {
    authgear.fetchUserInfo { userInfoResult in
        // sessionState is now up to date
        // it will change to .noSession if the session is invalid
        let sessionState = authgear.sessionState

        switch userInfoResult {
        case let .success(userInfo):
            // read the userInfo if needed
        case let .failure(error):
            // failed to fetch user info
            // the refresh token maybe expired or revoked
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
// value can be NO_SESSION or AUTHENTICATED
// After authgear.configure, it only reflect SDK local state.
SessionState sessionState = authgear.getSessionState();

if (sessionState == SessionState.AUTHENTICATED) {
    authgear.fetchUserInfo(new OnFetchUserInfoListener() {
        @Override
        public void onFetchedUserInfo(@NonNull UserInfo userInfo) {
            // sessionState is now up to date
            // read the userInfo if needed
        }

        @Override
        public void onFetchingUserInfoFailed(@NonNull Throwable throwable) {
            // sessionState is now up to date
            // it will change to NO_SESSION if the session is invalid
        }
    });
}
```
{% endtab %}
{% endtabs %}

* **Javascript Only**. Authgear SDK provides `fetch` function for you to call your application server. The `fetch` function will include Authorization header in your application request, and handle refresh access token automatically. `authgear.fetch` implement [fetch](https://fetch.spec.whatwg.org/). If you are using another networking library and want to include the Authorization header yourself. You can skip this and go to the next step.

{% tabs %}
{% tab title="Javascript" %}
```javascript
authgear
    .fetch("YOUR_SERVER_URL")
    .then(response => response.json())
    .then(data => console.log(data));
```
{% endtab %}
{% endtabs %}

* You will need to include the Authorization header in your application request, you can skip this if you are using `authgear.fetch` in Javascript SDK. Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make the network call only if the access token has expired. Include the access token into the Authorization header of your application request.

{% tabs %}
{% tab title="Javascript" %}
```java
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
{% endtab %}

{% tab title="iOS" %}
```swift
authgear.refreshAccessTokenIfNeeded() { result in
    switch result {
    case .success():
        // access token is ready to use
        // accessToken can be empty
        // it will be empty if user is not logged in or session is invalid

        // include Authorization header in your application request
        if let accessToken = authgear.accessToken {
            // example only, you can use your own networking library
            var urlRequest = URLRequest(url: "YOUR_SERVER_URL")
            urlRequest.setValue(
                "Bearer \(accessToken)", forHTTPHeaderField: "authorization")
            // ... continue making your request
        }
    case let .failure(error):
        // failed to refresh access token
        // the refresh token maybe expired or revoked
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
// Suppose we are preparing an http request in a background thread.

// Setting up the request, e.g. preparing a URLConnection

try {
    authgear.refreshAccessTokenIfNeededSync();
} catch (OauthException e) {
    // The token is expired.
}
String accessToken = authgear.getAccessToken();
if (accessToken == null) {
    // The user is not logged in, or the token is expired.
    // It is up to the caller to decide how to handle this situation. Typically, the request could be aborted
    // immediately as the response would be 401 anyways.
    return;
}

HashMap<String, String> headers = new HashMap<>();
headers.put("authorization", "Bearer " + accessToken);

// Submit the request with the headers...
```
{% endtab %}
{% endtabs %}

* Handle the case that the session is revoked after SDK obtained the access token. In this case, the app will call your application server with the access token. But your application server will find that the access token is invalid, by checking the [resolve headers](../getting-started/backend-integration/nginx.md). Depending on your application flow, you may want to show your user login page again or reset the SDK `sessionState` to `NO_SESSION` locally. To clear the `sessionState`, you can use `clearSessionState` function.

{% tabs %}
{% tab title="Javascript" %}
```javascript
// example only
// if your application server return HTTP status code 401 for unauthorized request
async function fetchAppServer() {
    var response = await authgear.fetch("YOUR_SERVER_URL");
    if (response.status === 401) {

        // if you want to clear the session state locally, call clearSessionState
        // `authgear.sessionState` will become `NO_SESSION` after calling
        await authgear.clearSessionState();
        throw new Error("user session invalid");
    }
    // ...
}
```
{% endtab %}

{% tab title="iOS" %}
```swift
// example only
// if your application server return HTTP status code 401 for unauthorized request
if let response = response as? HTTPURLResponse {
    if response.statusCode == 401 {

        // if you want to clear the session state locally, call clearSessionState
        // `authgear.sessionState` will become `.noSession` after calling
        authgear.clearSessionState { result in
            switch result {
            case .success():
                // clear SDK session state locally
                // `authgear.sessionState` becomes `.noSession`
            case let .failure(error):
                // failed to clear session state
            }
        }
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
// example only
// if your application server return HTTP status code 401 for unauthorized request
responseCode = httpConn.getResponseCode();
if (responseCode != HttpURLConnection.Unauthorized) {

    // if you want to clear the session state locally, call clearSessionState
    // ` authgear.getSessionState()` will become `NO_SESSION` after calling
    authgear.clearSessionState();
}
```
{% endtab %}
{% endtabs %}

