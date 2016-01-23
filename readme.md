# Falcor Demo

This is the code for Auth0's [Getting Started With Falcor](https://auth0.com/blog/2015/08/28/getting-started-with-falcor/) blog post. In the tutorial, we cover how to:

1. Setup a Falcor model on the client and prime it with data
2. Use Falcor's JSON Graph to avoid having duplicate data
3. Setup a Falcor Router with Falcor Express
4. Do a basic search using a Falcor route

## Installation

Clone the repo and then run:

    npm install
    node index.js

Navigate to localhost:3000

## Important Snippets

A model cache can be setup on the front end with some data by using a `new falcor.Model`

```js
var model = new falcor.Model({
  cache: {
    events: [
      {
        name: "ng-conf",
        description: "The world's best Angular Conference",
        stuff: "oh hey stuff",
        location: { city: "Salt Lake City", state: "Utah" }
      },
      {
        name: "React Rally",
        description: "Conference focusing on Facebook's React",
        location: { city: "Salt Lake City", state: "Utah" }
      },

...
```

We can then `get` some of the data by providing a set of JavaScript paths to the specific data we are looking for.

```js
model
  // To get the values on the "location" object, we need to pass the paths for the
  // keys on that object
  .get(["events", {from: 0, to: 2}, ["name", "description", "location"],["city", "state"]])
  .then(function(response) {
    document.getElementById("event-data").innerHTML = JSON.stringify(response, null, 2);
  });
```

We can then move the data over the server to be served with `falcor-express`. We need to have `express` serve a `model.json` endpoint and return a `new Router`.

```js
// We setup a model.json endpoint and pass it a dataSourceRoute which
// allows us to serve a router. Various route requests can be sent to the
// router to request whatever data is required
app.use('/model.json', falcorExpress.dataSourceRoute(function(req, res) {
  return new Router([
    {
      // Our route needs to match a pattern of integers that
      // are used as eventIds
      route: "events[{integers:eventIds}]['name', 'description', 'location']",
      get: function(pathSet) {
        
        var results = [];

        // Above we specified an eventIds identifier that is an
        // array of ids which we can loop over
        pathSet.eventIds.forEach(function(eventId) {

          // Next, an array of key names that map is held at pathSet[2]
          pathSet[2].map(function(key) {

            // We find the event the cooresponds to the current eventId
            var eventRecord = eventsData.events[eventId];

            // Finally we push a path/value object onto
            // the results array
            results.push({
              path: ['events', eventId, key], 
              value: eventRecord[key]
            });
          });          
        });

        return results;

...
```

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amont others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a Free Auth0 Account

1. Go to [Auth0](https://auth0.com) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for more info.
