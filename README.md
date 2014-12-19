
# passport-socketio-redis Library for socket.io 1.x and express.js 4.x
> Share user information from a [passport.js](http://passportjs.org) stored in [Connect-Redis](https://www.npmjs.com/package/connect-redis) with [socket.io](http://socket.io) connection.

## Installation

```
$ npm install passport-socketio-redis
```

## Example usage


```javascript

// Create http server (Based on express.js)

var express = require('express');
var app = express();
var server = app.listen(config.httpOptions.port, function(){
    console.log('Server is listnening on port %d', server.address().port);
});
var cookieParser = require('cookie-parser');
var redis = require("redis").createClient();
var passport = require('passport');
var session = require('express-session');
var RedisStore = require('connect-redis')(session);

var socketioRedis = require("passport-socketio-redis");


// When configure your session for express use options like this.
app.use(session({
    secret:"secret key", // Keep your secret key
    key:"connect.sid", 
    store:new RedisStore({
        host: 'localhost',
        port: 6379,
        client: redis
    }))
);

// Use passport
app.use(passport.initialize());
app.use(passport.session());

// Initialize socket.io 
var io = require("socket.io")(server);

io.use(socketioRedis.authorize({
    passport:passport,
    cookieParser: cookieParser,
    key:         'connect.sid',       
    secret:      'secret key',    
    store:       new RedisStore({ host: 'localhost', port: 6379, client: redis }),
    success:     authorizeSuccess,  // *optional* callback on success - read more below
    fail:        authorizeFail     // *optional* callback on fail/error - read more below
}));

function authorizeSuccess(data, accept)
{
    console.log('Authorized success');
    accept();
}

function authorizeFail(data, message, error, accept)
{
    if(error)
        accept(new Error(message));
}
```

## passport-socketio-redis - Options

```

### `cookieParser` [function] **required**:
You have to provide your cookieParser from express: `express.cookieParser`

### `key` [string] **optional**:
Defaults to `'connect.sid'`. But you're always better of to be sure and set your own key. Don't forget to also change it in your `express.session()`:
`app.use(session({ key: 'your.sid-key' }));`

### `secret` [string] **optional**:
As with `key`, also the secret you provide is optional. *But:* be sure to have one. That's always safer. You can set it like the key:
`app.use.session({ secret: 'pinkie ate my cupcakes!' }));`

### `passport` [function] **optional**:
Defaults to `require('passport')`. If you want, you can provide your own instance of passport for whatever reason.


```

## `socket.request.user` 
This property is always available from inside a `io.on('connection')` handler. If the user is authorized via passport, you can access all the properties from there.
for example:
```javascript
io.on('connection', function(socket){
    console.log(socket.request.user);
}
```
**Plus** you have the `socket.request.user.logged_in` property which tells you whether the user is currently authorized or not.

## Additional methods

### `passportSocketIo.filterSocketsByUser`
This function gives you the ability to filter all connected sockets via a user property. Needs two parameters `function(io, function(user))`. Example:
```javascript
passportSocketIo.filterSocketsByUser(io, function(user){
  return user.gender === 'female';
}).forEach(function(socket){
  socket.emit('messsage', 'hello, woman!');
});
```

### Clientside:

You have to provide the session-cookie. If you haven't set a name yet, do it like this: `app.use(session({ key: 'your.sid-key' }));`
For example, my Angular.js implementation
```javascript
appServices.factory('socket', ['$rootScope',function ($rootScope) {
    var socket = io.connect(); // Connect to server

    return {
        on: function (eventName, callback) {
            socket.on(eventName, function () {
                var args = arguments;
                $rootScope.$apply(function () {
                    callback.apply(socket, args);
                });
            });
        },
        emit: function (eventName, data, callback) {
            socket.emit(eventName, data, function () {
                var args = arguments;
                $rootScope.$apply(function () {
                    if (callback) {
                        callback.apply(socket, args);
                    }
                });
            })
        }
    };
}]);
```

### Serverside:
Nope, there's nothing to do on the server side. Just be sure that the cookies names match.


## License
Licensed under the MIT-License.
2012-2015 Filip Lukac.

## Library base on [passport.socketio](https://github.com/jfromaniello/passport.socketio)
