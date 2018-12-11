# express-hot-shots

[StatsD](https://github.com/etsy/statsd/) route monitoring middleware for 
[Connect](https://github.com/senchalabs/connect)/[Express](https://github.com/visionmedia/express).
This middleware can be used either globally or on a per-route basis (preferred)
and sends status codes and response times to StatsD.

Forked from [uber/express-statsd](https://github.com/uber/express-statsd) for usage in [Spectrum](https://github.com/withspectrum/spectrum).

## Installation

``` bash
npm install express-hot-shots
```

## Usage

An example of an express server with express-hot-shots:

``` js
var express = require('express');
var statsd = require('express-hot-shots');
var app = express();

app.use(statsd());

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000);
```

By default, the middleware will send `status_code` and `response_time` stats
for all requests. For example, using the created server above and a request to
`http://localhost:3000/`, the following stats will be sent:

```
status_code.200:1|c
response_time:100|ms
```

### Per route example

However, it's **highly recommended** that you set `req.statsdKey` which
will be used to namespace the stats. Be aware that stats will only be logged
once a response has been sent; this means that `req.statsdKey` can be
set even after the express-hot-shots middleware was added to the chain. Here's an 
example of a server set up with a more specific key:

``` js
var express = require('express');
var expressStatsd = require('express-hot-shots');
var app = express();

function statsd (path) {
  return function (req, res, next) {
    var method = req.method || 'unknown_method';
    req.statsdKey = ['http', method.toLowerCase(), path].join('.');
    next();
  };
}

app.use(expressStatsd());

app.get('/', statsd('home'), function (req, res) {
  res.send('Hello World!');
});

app.listen(3000);
```

A GET request to `/` on this server would produce the following stats:

```
http.get.home.status_code.200:1|c
http.get.home.response_time:100|ms
```

### Tags

You can set the tags of the metrics with the `req.statsdTags` property.

```JS

function statsd (path) {
  return function (req, res, next) {
    var method = req.method || 'unknown_method';
    req.statsdKey = ['http', method.toLowerCase(), path].join('.');
    req.statsdTags = {
      server: process.env.SERVER_NAME,
    }
    next();
  };
}
```

These will be sent with both the response time and status code metrics.

### Plain http example

This module also works with any `http` server

```js
var http = require('http');
var expressStatsd = require('express-hot-shots');

var monitorRequest = expressStatsd();

http.createServer(function (req, res) {
    monitorRequest(req, res);

    // do whatever you want, framework, library, router
    res.end('hello world');
}).listen(3000);
```

## Options

``` js
expressStatsd(options);
```

- **options** `Object` - Container for settings
  - **hotShots** `Object` - The hotShots options
  - **requestKey** `String` - The key on the `req` object at which to grab
the key for the statsd logs. Defaults to `req.statsdKey`.
