anyFetch api wrapper for Node.js
================================
[![Build Status](https://travis-ci.org/AnyFetch/anyfetch.js.png?branch=master)](https://travis-ci.org/AnyFetch/anyfetch.js)
[![Dependency Status](https://gemnasium.com/AnyFetch/anyfetch.js.png)](https://gemnasium.com/AnyFetch/anyfetch.js)
[![Coverage Status](https://coveralls.io/repos/AnyFetch/anyfetch.js/badge.png?branch=master)](https://coveralls.io/r/AnyFetch/anyfetch.js?branch=master)
[![NPM version](https://badge.fury.io/js/anyfetch.png)](http://badge.fury.io/js/anyfetch)


> You'll only be interested in this package if you want to create client applications for [AnyFetch](http://anyfetch.com).

> If you're interested in creating a Hydrater or a Provider, you may find [AnyFetch Hydrater Library](https://github.com/AnyFetch/hydrater-boilerplate) and [AnyFetch Provider Library](https://github.com/AnyFetch/anyfetch-provider.js) more high level, and easier to work with.

This `npm` package makes communicating with the AnyFetch servers easy for clients. Check out the [**full API documentation**](http://developers.anyfetch.com/endpoints/).

## Basic usage example

```js
var AnyFetch = require('../lib/index.js');

var anyfetchBasic = new AnyFetch('LOGIN', 'PASSWORD');
// OR
var anyfetch = new AnyFetch('TOKEN');

anyfetch.getCurrentUser(function(err, user) {
  console.log('Hello, my name is ' + user.name);
};
```

## Access authentication

Both `Basic` and `Bearer` authentication schemes are supported. Note that some endpoints are only accessible using `Bearer` auth. The `getToken` method makes it easy to retrieve a token from the user's credentials.
Please note: AnyFetch delivers long lived `access_token`, so you don't need to use a `refresh_token`.

```js
var Anyfetch = require('../lib/index.js');

var anyfetchBasic = new Anyfetch('LOGIN', 'PASSWORD');

// Retrieve token from credentials (GET /token)
anyfetchBasic.getToken(function(err, res) {
  if(err) {
    throw err;
  }
  
  anyfetch = new Anyfetch(res.body.token);
  // We now access the Fetch API using Bearer authentication
};
```

## oAuth

The `getAccessToken` static function helps you obtain an `access_token` during the oAuth flow.

```js
Anyfetch.getAccessToken('APP_ID', 'APP_SECRET', 'OAUTH_VERIFICATION_CODE', function(err, accessToken) {
  var anyfetch = new Anyfetch(accessToken);
});
```

## Basic endpoint to function mappings

This library provides a function per [API endpoint](http://developers.anyfetch.com/endpoints/). We adopt the following naming convention:

```js
verbEndpointName(function(error, result) {})
```

Callbacks are expected to be of the form: `function(err, result)`. Note that some endpoints do not yield any result (e.g. `POST /company/update`).

Examples:

- `getUsers(cb)` will call `GET /users`
- `postCompanyUpdate(cb)` will call `POST /company/update`
- `deleteCompanyReset(cb)` will call `DELETE /company/reset`
- `deleteToken(cb)` will call `DELETE /token`

Some functions expect an `id` or `identifier`:

- `getDocumentById(id, cb)` will call `GET /documents/{id}`
- `getDocumentByIdentifier(identifier, cb)` will call `GET /documents/identifier/{identifier}`

Some other endpoints are expressed relative to a document. For example, `GET /documents/{id}/raw` refers to the document with id `{id}`.
For the sake of clarity, we provide the following two-steps call syntax:

- `getDocumentById(id).getSimilar(cb)` will call `GET /documents/{id}/similar`
- `getDocumentById(id).getRaw(cb)` will call `GET /documents/{id}/raw`
- `getDocumentById(id).postFile(cb)` will call `POST /documents/{id}/file`

Note that the first function **does not take any callback**. It is simply responsible for building the first part of the request, which is then carried out when calling the sub-function.

## Utility functions

`anyfetch.js` provides higher level utility functions. They cover classic use-cases that would otherwise require several API calls. When possible, calls are grouped in a single batch call.

### Batch request

Make several `GET` calls in a single request. It takes a map associating the endpoint to its parameters.

```js
var pages = {
  '/users': null,
  '/documents': {
    search: 'Marc'
  }
};
self.batch(pages, cb);
```

### Create a subcompany

When [creating a subcompany](http://developers.anyfetch.com/endpoints/#subcompanies-subcompanies-post), we usually want to create its first admin, and migrate it into the new subcompany. The function `createSubcompanyWithAdmin` allows you to do this automatically.
The created user **will be** an admin.

```js
var subcompany = {
  name: 'the_fake_subcompany',
  hydraters: [
    'http://plaintext.hydrater.anyfetch.com/hydrate',
    'http://pdf.hydrater.anyfetch.com/hydrate',
  ]
};
var admin = {
  email: 'thechuck@norris.com',
  name: 'Chuck Norris',
  password: 'no_need'
};
anyfetch.createSubcompanyWithAdmin(subcompany, admin, function(err, company) {
  console.log('Company ' + company.id + ' has been created');
});
```

### Get current user

This function allows you to retrieve the user's info from its credentials (login / password or token).

```js
anyfetch.getCurrentUser(function(err, user) {
  console.log('Hello, my name is ' + user.name);
};
```

### Get document(s) with info

When developping a front-end for the AnyFetch API, it's common to need the `document_type` and `provider` of a particular document. This function allows you to do this in one call.

```js
anyfetch.getDocumentWithInfo(documentId, function(err, doc) {
  document.log('This document is a ' + doc.document_type.name + ' and has been provided by ' + doc.provider.name);
});
```

Related:
- `getDocumentByIdentifierWithInfo(identifier, cb)` is similar but finds the document by its `identifier` instead of its `id`
- `getDocumentsWithInfo(params, cb)` returns the documents matched by the request expressed in `params`

## Test framework

`anyfetch.js` provides a ready-to-run mock server based on Restify. It may be useful to test apps that use the AnyFetch API.

The mock server is created with `Anyfetch.createMockServer()` and started with `server.listen(port, cb)`. It is a simple [Restify server](http://mcavage.me/node-restify/).
Once the server is running, override the AnyFetch API host to make it point to your `localhost` with `anyfetch.setApiHost(apiHost)`.

**Example**: starting the mock server on port 1337

```js
var AnyFetch = require('anyfetch.js');
server = Anyfetch.createMockServer();

var port = 1337;
var apiHost = 'http://localhost:' + port;
server.listen(port, function() {
  console.log('Anyfetch mock server running on ' + apiHost);
  anyfetch.setApiHost(apiHost);

  done();
});
```

### Mocks

The mock server will serve static JSON files from the folder `node_modules/anyfetch/lib/test-server/mocks`. You can tailor them to your need. You can generate those JSON files at any time by:

1. Setting the `LOGIN` and `PASSWORD` environment variables to your Fetch API credentials
2. Running `node node_modules/anyfetch/bin/make-mocks.js`
