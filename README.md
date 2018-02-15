# Hapi 17 compatible swagger-hapi

This project is a [hapi.js](https://hapijs.com/) 17 compatible fork of [swagger-hapi](https://github.com/apigee-127/swagger-hapi). It relies on the associated Hapi 17 [compatible fork](https://github.com/DEFRA/swagger-node-runner) of the [swagger-node-runner](https://github.com/theganyo/swagger-node-runner) project.

## Migrating an existing swagger-hapi project
### Update dependencies
Within package.json:
* Replace the reference to the npm swagger-hapi package with a reference to the current version of the npm defra-swagger-hapi-package.
* Upgrade the hapi dependency to ^17.0.0
### Perform upstream upgrades
Perform the upgrades documented by the original [swagger-node-runner 0.6.0 or above](https://github.com/theganyo/swagger-node-runner/releases/tag/v0.6.0) if required by your project.
### Migrate bootstrap code
Migrate the bootstrap code in app.js to ensure compatibility with Hapi 17. For example,
```
'use strict'
var SwaggerHapi = require('swagger-hapi');
var Hapi = require('hapi');
var app = new Hapi.Server();

module.exports = app; // for testing

var config = {
  appRoot: __dirname // required config
};

SwaggerHapi.create(config, function(err, swaggerHapi) {
  if (err) { throw err; }

  var port = process.env.PORT || 10010;
  app.connection({ port: port });
  app.address = function() {
    return { port: port };
  };

  app.register(swaggerHapi.plugin, function(err) {
    if (err) { return console.error('Failed to load plugin:', err); }
    app.start(function() {
      if (swaggerHapi.runner.swagger.paths['/hello']) {
        console.log('try this:\ncurl http://127.0.0.1:' + port + '/hello?name=Scott');
      }
    });
  });
});
```
could become 
```
'use strict';
const SwaggerHapi = require('defra-swagger-hapi');
const Hapi = require('hapi');
const port = process.env.PORT || 10010;
const options = {
  port: port
}
const app = new Hapi.Server(options)

module.exports = app // for testing

const config = {
  appRoot: __dirname, // required config
};

SwaggerHapi.create(config, function (err, swaggerHapi) {
  if (err) { throw err }
  app.address = function () {
    return { port: port };
  };

  async function init() {
    try {
      app.events.on('start', function() {
        if (swaggerHapi.runner.swagger.paths['/hello']) {
          console.log('try this:\ncurl http://127.0.0.1:' + port + '/hello?name=Scott');
        }
      });
      await app.register(swaggerHapi.plugin);
      await app.start();
    } catch (err) {
      return console.error('Failed to load plugin:', err)
    }
  }
  init();
})
```
If your bootstrap code has more reliance on legacy hapi.js code, please see the hapi.js website for guidance.
### Migrating other legacy hapi.js code
If non-bootstrap code requires migration to Hapi 17 please see the hapi.js website for guidance.

## Creating a new Hapi 17 compatible project
Forking the original [swagger-node](https://github.com/swagger-api/swagger-node) project to provide compatibility with Hapi 17 has been considered but decided against. If you need to create a new project compatible with Hapi 17, please use the original swagger-node project to generate the initial code base and then refactor the bootstrap code as for an existing project as described above.

## Contributing to this project

If you have an idea you'd like to contribute please log an issue.

All contributions should be submitted via a pull request.

## License

THIS INFORMATION IS LICENSED UNDER THE CONDITIONS OF THE OPEN GOVERNMENT LICENCE found at

<http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3>

The following attribution statement MUST be cited in your products and applications when using this information.

>Contains public sector information licensed under the Open Government license v3

### About the license

The Open Government Licence (OGL) was developed by the Controller of Her Majesty's Stationery Office (HMSO) to enable information providers in the public sector to license the use and re-use of their information under a common open licence.

It is designed to encourage use and re-use of information freely and flexibly, with only a few conditions.

### Compliance with the licencing terms of the original work

In accordance with the MIT licencing terms of the original work on which this work is based,
a copy of the original licence and copyright notice is included within the file ORIGINAL-LICENSE.
