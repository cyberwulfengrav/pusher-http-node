# Pusher node.js Server library

This is a node.js library for interacting with the Pusher REST API.

Registering at <http://pusher.com> and use the application credentails within your app as shown below.

## Installation
```
$ npm install pusher
```

## How to use

### Constructor

There are 3 ways to configure the client. First one is the most basic:

```javascript
var Pusher = require('pusher');

var pusher = new Pusher({
  appId: 'APP_ID',
  key: 'APP_KEY',
  secret: 'SECRET_KEY',
  scheme: 'SCHEME', // optional, defaults to http
  host: 'HOST', // optional, defaults to api.pusherapp.com
  port: PORT, // optional, defaults to 80 for http and 443 for https
});
```

For specific clusters, you can also use the cluster option:

```javascript
var Pusher = require('pusher');

var pusher = new Pusher({
  appId: 'APP_ID',
  key: 'APP_KEY',
  secret: 'SECRET_KEY',
  scheme: 'SCHEME', // optional, defaults to http
  cluster: 'CLUSTER', // will set the host to `api-CLUSTER.pusher.com`
  port: PORT, // optional, defaults to 80 for http and 443 for https
});
```

You can also specify auth and endpoint options by passing an URL:

```javascript
var Pusher = require('pusher');

var pusher = new Pusher({
  url: "SCHEME://APP_KEY:SECRET_KEY@HOST:PORT/apps/APP_ID"
});
```

This is useful for example on Heroku, which sets the PUSHER_URL environment
variable to such URL, if you have the Pusher addon installed.

#### Additional constructor options

There are a few additional options that can be passed to the constructor:

```javascript
var Pusher = require('pusher');

var pusher = new Pusher({
  // you can set other options in any of the 3 ways described above
  proxy: 'HTTP_PROXY_URL', // optional, URL to proxy the requests through
  timeout: TIMEOUT, // optional, timeout for all requests in milliseconds
  keepAlive: KEEP_ALIVE // optional, enables keep-alive, defaults to false
});
```

### Publishing/Triggering events

To trigger an event on one or more channels use the trigger function.

#### A single channel

```javascript
pusher.trigger('channel-1', 'test_event', { message: "hello world" });
```

#### Multiple channels

```javascript
pusher.trigger([ 'channel-1', 'channel-2' ], 'test_event', { message: "hello world" });
```

### Excluding event recipients

In order to avoid the person that triggered the event also receiving it the `trigger` function can take an optional `socketId` parameter. For more informaiton see: <http://pusher.com/docs/publisher_api_guide/publisher_excluding_recipients>.

```javascript
var socketId = '1302.1081607';

pusher.trigger(channel, event, data, socketId);
```

### Authenticating Private channels

To authorise your users to access private channels on Pusher, you can use the `auth` function:

```javascript
var auth = pusher.auth(socketId, channel);
```

For more information see: <http://pusher.com/docs/authenticating_users>

### Authenticating Presence channels

Using presence channels is similar to private channels, but you can specify extra data to identify that particular user:

```javascript
var channelData = {
	user_id: 'unique_user_id',
	user_info: {
	  name: 'Phil Leggetter'
	  twitter_id: '@leggetter'
	}
};
var auth = pusher.auth(socketId, channel, channelData);
```

The `auth` is then returned to the caller as JSON.

For more information see: <http://pusher.com/docs/authenticating_users>

### Application State

It's possible to query the state of the application using the `pusher.get` function.
```javascript
pusher.get({ path: path, params: params }, callback);
```
The `path` property identifies the resource that the request should be made to and the `params` property should be a map of additional querystring key and value pairs.

The following example provides the signature of the callback and an example of parsing the result:
```javascript
pusher.get({ path: '/channels', params: {} }, function(error, request, response) {
	if (response.statusCode === 200) {
		var result = JSON.parse(response.body);
		var channelsInfo = result.channels;
	}
});
```

#### Get list of channels in an application
```javascript
pusher.get({ path: '/channels', params: params }, callback);
```

Information on the optional `params` option property and the structure of the returned JSON is defined in the [REST API reference](http://pusher.com/docs/rest_api#method-get-channels).

#### Get single channel state
```javascript
pusher.get({ path: '/channels/[channel_name]', params: params }, callback);
```

Information on the optional `params` option property and the structure of the returned JSON is defined in the [REST API reference](http://pusher.com/docs/rest_api#method-get-channel).

#### Get list of users on a presence channel
```javascript
pusher.get({ path: '/channels/[channel_name]/users' }, callback);
```

The `channel_name` in the path must be a [presence channel](http://pusher.com/docs/presence). The structure of the returned JSON is defined in the [REST API reference](http://pusher.com/docs/rest_api#method-get-users).

### WebHooks

The library provides a simple helper for WebHooks, which can be accessed via Pusher objects:

```javascript
var webhook = pusher.webhook(request);
```

Requests must expose following fields:
- headers - object with request headers indexed by lowercase header names
- rawBody - string with the WebHook request body

Since neither Node.js nor express provide the body in the request, your application needs to read it and assign to the request object. See examples/webhook_endpoint.js for a simple webhook endpoint implementation using the express framework.

Headers object must contain following headers:
- x-pusher-key - application key, sent by Pusher
- x-pusher-signature - WebHook signature, generated by Pusher
- content-type - must be set to application/json, what Pusher does

After instantiating the WebHook object, you can use its following methods:

#### isValid

Validates the signature of the WebHook and returns a boolean. Your application should validate incoming webhooks,
otherwise they could be faked.

Accepts an optional parameter containing additional application tokens (useful e.g. during migrations):

```javascript
var webhook = pusher.webhook(request);
// will check only the key and secret assigned to the pusher object:
webhook.isValid();
// will also check two additional tokens:
webhook.isValid([{ key: "x1", secret: "y1" }, { key: "x2", secret: "y2" }]);
```

#### getData

Returns the parsed WebHook body. Throws an error if the content type is not `application/json` or the body is not valid JSON.

```javascript
// will return an object with the WebHook data
webhook.getData();
```

Please read [the WebHooks documentation](http://pusher.com/docs/webhooks) to find out what fields are included in the body.

#### getEvents

Returns events included in the WebHook as an array. Throws errors the same way as `getData`.

```javascript
// will return an array with the events
webhook.getEvents();
```

#### getTime

Returns the Date object for the time when the WebHook was sent from Pusher. Throws errors the same way as `getData`.

```javascript
// will return a Date object
webhook.getTime();
```

## Tests

The tests run using [Mocha](http://visionmedia.github.io/mocha/). Make sure
you've got all required modules installed:

    npm install

### Unit tests

You can run unit tests without setting up a Pusher app:

    node_modules/.bin/mocha tests/unit/**/*.js

### Full test suite

In order to run the full test suite, first you need a Pusher app. When starting
mocha, you need to set the PUSHER_URL environment variable to contain your
app credentials, like following:

    `PUSHER_URL='http://KEY:SECRET@api.pusherapp.com/' node_modules/.bin/mocha tests/**/*.js`

## Credits

This library is based on the work of:
* Christian Bäuerlein and his library pusher.
* Jaewoong Kim and the node-pusher library.

## License

This code is free to use under the terms of the MIT license.
