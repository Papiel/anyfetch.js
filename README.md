anyFetch api wrapper for Node.js
================================
[![Build Status](https://travis-ci.org/AnyFetch/anyfetch.js.png?branch=master)](https://travis-ci.org/AnyFetch/anyfetch.js)
[![Dependency Status](https://gemnasium.com/AnyFetch/anyfetch.js.png)](https://gemnasium.com/AnyFetch/anyfetch.js)
[![Coverage Status](https://coveralls.io/repos/AnyFetch/anyfetch.js/badge.png?branch=master)](https://coveralls.io/r/AnyFetch/anyfetch.js?branch=master)
[![NPM version](https://badge.fury.io/js/anyfetch.png)](http://badge.fury.io/js/anyfetch)


> You'll only be interested in this package if you want to create client applications for [AnyFetch](http://anyfetch.com).

> If you're interested in creating a Hydrater or a Provider, you may find [AnyFetch Hydrater Library](https://github.com/AnyFetch/hydrater-boilerplate) and [AnyFetch Provider Library](https://github.com/AnyFetch/anyfetch-provider.js) more high level, and easier to work with.

This npm package makes communicating with the AnyFetch servers easy for clients. Check out the [**full API documentation**](http://developers.anyfetch.com/endpoints/).

Please note: AnyFetch delivers long lived `access_token`, so you don't need to use a `refresh_token`.

## Example

```js
var AnyFetch = require('../lib/index.js');

var anyfetchBasic = new AnyFetch('LOGIN', 'PASSWORD');
// OR
var anyfetch = new AnyFetch('TOKEN');

anyfetch.getCurrentUser(function(err, user) {
  console.log('Hello, my name is ' + user.name);
};
```

## Basic function / endpoint mappings

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

We use specific names when passing parameters:

- `getDocumentById(id, cb)` will call `GET /documents/{id}`
- `getDocumentByIdentifier(identifier, cb)` will call `GET /documents/identifier/{identifier}`

Some endpoints are expressed relative to a document. For the sake of clarity, we provide the following call syntax:

- `getDocumentById(id).getSimilar(cb)` will call `GET /documents/{id}/similar`
- `getDocumentById(id).getRaw(cb)` will call `GET /documents/{id}/raw`
- `getDocumentById(id).postFile(cb)` will call `POST /documents/{id}/file`

Note that the first function **does not take any callback**. It is simply responsible for building the first part of the request, which is then carried out when calling the second part.

## Helper functions

`anyfetch.js` provides higher level functions.

## Test framework

`anyfetch.js` provides a ready-to-run mock server based on Restify. It may be useful to test apps that use the AnyFetch API.

The mock server is created with `Anyfetch.createMockServer()` and started with `server.listen(port, cb)`. It is a simple [Restify server](http://mcavage.me/node-restify/).
Once the server is running, override the AnyFetch API host to make it point to your `localhost` with `anyfetch.setApiHost(apiHost)`.

### Example

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
