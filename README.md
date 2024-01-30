<p align="center">
  <img height="300px" src="https://authress.io/static/images/media-banner.png" alt="Authress media banner">
</p>

# Authress Login SDK for UIs

<p align="center">
    <a href=https://www.npmjs.com/package/@authress/login" alt="Authress SDK on npm"><img src="https://badge.fury.io/js/@authress%2Flogin.svg"></a>
    <a href="./LICENSE" alt="Apache-2.0"><img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg"></a>
    <a href="https://authress.io/community" alt="authress community"><img src="https://img.shields.io/badge/Community-Authress-fbaf0b.svg"></a>
</p>

The Authress Universal Login SDK for javascript app websites and user identity authentication. Used to integrate with the authentication as a service provider Authress at https://authress.io.


## Usage

```sh
npm install @authress/login
```

Then required the package:
```js
import { LoginClient } from '@authress/login';
```

### Troubleshooting usage
Troubles with esbuild or another bundler, checkout the [build tools troubleshooting page](./docs/troubleshooting.md).

## Getting Started

### Part 0: Setup Authress Login
You'll want to create:
* at least one provider connection - https://authress.io/app/#/manage?focus=connections
* an application which represents your web app - https://authress.io/app/#/manage?focus=applications

### Part 1: Web App UI

On every route change check to see if the user exists, and if they don't redirect them to a login prompt.
```js
import { LoginClient } from '@authress/login';

// What is my applicationId => https://authress.io/app/#/manage?focus=applications
// What is my authressApiUrl? => https://authress.io/app/#/setup?focus=domain
const loginClient = new LoginClient({ authressApiUrl: 'https://login.application.com', applicationId: 'YOUR_APPLICATION_ID' });
const isUserLoggedIn = await loginClient.userSessionExists();
if (!isUserLoggedIn) {
  window.location.assign('/login');
}
```
In your app's login screen when the user selects how they would like to login, direct them there. And also specify where you would like Authress to redirect the user to after login. By default this is the user's current location.
```js
await loginClient.authenticate({ connectionId: 'SELECTED_CONNECTION_ID', redirectUrl: window.location.href });
// Or if you know which tenant the user wants to log in with:
await loginClient.authenticate({ tenantLookupIdentifier: 'tenant-subdomain.app.com', redirectUrl: window.location.href });
return;
```

Then get the user access token associated with the user's login to be used for authorization in the next part:
```js
const userToken = await loginClient.ensureToken();
```

### Part 2: User Authentication in Service APIs
To authenticate the user in your service API, the token generated by the library in `Part 1` needs to be passed to your service. The standard is pulling it from the loginClient, and putting it into an `Authorization` header:
```js
const userToken = await loginClient.ensureToken();
const result = await fetch('https://api.application.com/v1/route', {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${userToken}`
  }
})
```

On the service API side, pull in the Authress service client companion library, and then verify the access token.

* First install the companion library: `npm install authress-sdk`
* Then verify the incoming tokens from the Authorization header:

```js
const { TokenVerifier } = require('authress-sdk');

try {
  // Grab authorization token from the request header, the best way to do this will be framework specific.
  const userToken = request.headers['Authorization'];
  // Specify your custom domain for tokens. Configurable at https://authress.io/app/#/manage?focus=applications
  const userIdentity = await TokenVerifier('https://login.application.com', userToken);
} catch (error) {
  console.log('User is unauthorized', error);
  return { statusCode: 401 };
}
```

### Platform Extension Login
The goal of the platform extension is to make it easy for your platform extension developers to login with Authress. Embed the `ExtensionClient` in your javascript UI SDK, and pass in the `extensionId`.

```js
const { ExtensionClient } = require('@authress/login');

// What is my custom Domain? => https://authress.io/app/#/setup?focus=domain
// What is my extensionId => https://authress.io/app/#/manage?focus=extensions
const extensionClient = new ExtensionClient('https://login.application.io', extensionId);

// redirectUrl is where the extension would like to return the user to after login
// * This method will redirect the user to the Authress Login UI screen with an auth code
const { accessToken } = await extensionClient.login(redirectUrl);

// .... After login the user is redirected to the redirectUrl
// * So try the login again:
const { accessToken } = await extensionClient.login(redirectUrl);

// * Or get the user claims from the token
await userData = await this.getUserIdentity();
```

## Advanced
Curious exactly how these methods work and when they should be used? We have some advanced guidance available for each of the methods on the:

[Method documentation](./docs/advanced.md)

## Contributing

If you are interested in contributing to the @authress/login SDK, feel fere to read the [contribution.md](./docs/contibution.md) guide.