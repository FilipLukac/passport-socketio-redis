
# passport-socketio-redis Library for socket.io 1.x and express.js 4.x
> Share user information from a [passport.js](http://passportjs.org) stored in [Connect-Redis](https://www.npmjs.com/package/connect-redis) with [socket.io](http://socket.io) connection.


## Installation

```
$ npm install passport-socketio-redis
```

## Example usage


```javascript

// Create http server (Base on express.js)

var express = require('express');
var app = express();
var server = app.listen(config.httpOptions.port, function(){
    console.log('Server is listnening on port %d', server.address().port);
});
var cookieParser = require('cookie-parser');
var redis = require("redis").createClient();
var passport = require('passport');

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
var session = require('express-session');
var RedisStore = require('connect-redis')(session);



io.use(passportSocketIo.authorize({
    passport:passport,
    cookieParser: cookieParser,
    key:         'connect.sid',       
    secret:      'Lebq8zzuQe3MS9H',    // the session_secret to parse the cookie
    store:       new RedisStore({ host: 'localhost', port: 6379, client: redis }),
    success:     authorizeSuccess,  // *optional* callback on success - read more below
    fail:        onAuthorizeFail     // *optional* callback on fail/error - read more below
}));

function onAuthorizeSuccess(data, accept){
    console.log('Authorized success');
    accept();
}

function onAuthorizeFail(data, message, error, accept){

    f(error)
        accept(new Error(message));

}
```

## passport-socketio-redis - Options

```

### `cookieParser` [function] **required**:
You have to provide your cookieParser from express: `express.cookieParser`

### `key` [string] **optional**:
Defaults to `'connect.sid'`. But you're always better of to be sure and set your own key. Don't forget to also change it in your `express.session()`:
`app.use(express.session({ key: 'your.sid-key' }));`

### `secret` [string] **optional**:
As with `key`, also the secret you provide is optional. *But:* be sure to have one. That's always safer. You can set it like the key:
`app.use(express.session({ secret: 'pinkie ate my cupcakes!' }));`

### `passport` [function] **optional**:
Defaults to `require('passport')`. If you want, you can provide your own instance of passport for whatever reason.

### `success` [function] **optional**:
Callback which will be called everytime a *authorized* user successfuly connects to your socket.io instance. **Always** be sure to accept/reject the connection.
For that, there are two parameters: `function(data[object], accept[function])`. `data` contains all the user-information from passport.
The second parameter is for accepting/rejecting connections. Use it like this if you use socket.io under 1.0:
```javascript
// accept connection
accept(null, true);

// reject connection (for whatever reason)
accept(null, false);


```

And like this if you use the newest version of socket.io@1.X
```javascript
// accept connection
accept();

// reject connection (for whatever reason)
accept(new Error('optional reason'));


```


### `fail` [function] **optional**:
The name of this callback may be a little confusing. While it is called when a not-authorized-user connects, it is also called when there's a error.
For debugging reasons you are provided with two additional parameters `function(data[object], message[string], error[bool], accept[function])`: (socket.io @ < 1.X)
```javascript
/* ... */
function onAuthorizeFail(data, message, error, accept){
  // error indicates whether the fail is due to an error or just a unauthorized client
  if(error){
    throw new Error(message);
  } else {
    console.log(message);
    // the same accept-method as above in the success-callback
    accept(null, false);
  }
}

// or
// This function accepts every client unless there's an error
function onAuthorizeFail(data, message, error, accept){
  console.log(message);
  accept(null, !error);
}
```

Socket.io@1.X:
```javascript
function onAuthorizeFail(data, message, error, accept){
  // error indicates whether the fail is due to an error or just a unauthorized client
  if(error)  throw new Error(message);
  // send the (not-fatal) error-message to the client and deny the connection
  return accept(new Error(message));
}

// or
// This function accepts every client unless there's an critical error
function onAuthorizeFail(data, message, error, accept){
  if(error)  throw new Error(message);
  return accept();
}
```


You can use the `message` parameter for debugging/logging/etc uses.

## `socket.handshake.user` (prior to v1)
This property was removed in v1. See `socket.request.user`

## `socket.request.user` (as of v1)
This property is always available from inside a `io.on('connection')` handler. If the user is authorized via passport, you can access all the properties from there.
**Plus** you have the `socket.request.user.logged_in` property which tells you whether the user is currently authorized or not.

**Note:** This property was named socket.handshake.user prior to v1

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

## CORS-Workaround:
If you happen to have to work with Cross-Origin-Requests (marked by socket.io v0.9 as `handshake.xdomain` and by socket.io v1.0 as `request.xdomain`) then here's a workaround:

### Clientside:
You have to provide the session-cookie. If you haven't set a name yet, do it like this: `app.use(express.session({ key: 'your.sid-key' }));`
```javascript
// Note: ther's no readCookie-function built in.
// Get your own in the internetz
socket = io.connect('//' + window.location.host, {
  query: 'session_id=' + readCookie('your.sid-key')
});
```

### Serverside:
Nope, there's nothing to do on the server side. Just be sure that the cookies names match.


## Notes:
* Does **NOT** support cookie-based sessions. eg: `express.cookieSession`
* If the connection fails, check if you are requesting from a client via CORS. Check `socket.handshake.xdomain === true` (`socket.request.xdomain === true` with socket.io v1) as there are no cookies sent. For a workaround look at the code above.


## Contribute
You are always welcome to open an issue or provide a pull-request!
Also check out the unit tests:
```bash
npm test
```

## License
Licensed under the MIT-License.
2012-2013 José F. Romaniello.
