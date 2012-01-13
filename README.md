
<img src="https://github.com/flatiron/union/raw/master/union.png" />

# Synopsis
A hybrid streaming middleware kernel backwards compatible with connect.

# Motivation
The advantage to streaming middlewares is that they do not require buffering the entire stream in order to execute their function.

# Status

[![Build Status](https://secure.travis-ci.org/flatiron/union.png)](http://travis-ci.org/flatiron/union)

# Installation
There are a few ways to use `union`. Install the library using npm. You can add it to your `package.json` file as a dependancy

```bash
  $ [sudo] npm install union
```

## Usage
Union's request handling is [connect](https://github.com/senchalabs/connect)-compatible, meaning that all existing connect middlewares should work out-of-the-box with union.

In addition, the response object passed to middlewares listens for a "next" event, which is equivalent to calling `next()`. Flatiron middlewares are written in this manner, meaning they are not reverse-compatible with connect.

### A simple case

``` js
var fs = require('fs'),
    union = require('../lib'),
    director = require('director'),
    favicon = require('./middleware/favicon');

var router = new director.http.Router();

var server = union.createServer({
  before: [
    favicon('./favicon.png'),
    function (req, res) {
      var found = router.dispatch(req, res);
      if (!found) {
        res.emit('next');
      }
    }
  ]
});

router.get(/foo/, function () {
  this.res.writeHead(200, { 'Content-Type': 'text/plain' })
  this.res.end('hello world\n');
});

router.post(/foo/, { stream: true }, function () {
  var req = this.req,
      res = this.res,
      writeStream;
      
  writeStream = fs.createWriteStream(Date.now() + '-foo.txt');
  req.pipe(writeStream);
  
  writeStream.on('close', function () {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('wrote to a stream!');
  });
});

server.listen(9090);
console.log('union with director running on 9090');
```

# API

## `union` Static Members

### createServer(options)
The `options` object is required. Options include:

```
  function createServer(options)
  @param options {Object} an object literal that represents the configuration for the server.
    
    @option before {Array} The `before` value is an array of middlewares, which are used to route and serve incoming requests. For instance, in the example, `favicon` is a middleware which handles requests for `/favicon.ico`.

    @option after {Array} The `after` value is an array of stream filters, which are applied after the request handlers in `options.before`. Stream filters inherit from `union.ResponseStream`, which implements the Node.js core streams api with a bunch of other goodies.

    @option limit {Object} (optional) A value, passed to internal instantiations of `union.BufferedStream`.

    @option https {Object} (optional) A value that specifies the certificate and key necessary to create an instance of `https.Server`.

    @option headers {Object} (optional) An object representing a set of headers to set in every outgoing response

```

```js
var server = union.createServer({
  before: [
    favicon('./favicon.png'),
    function (req, res) {
      var found = router.dispatch(req, res);
      if (!found) {
        res.emit('next');
      }
    }
  ]
});
```

An example of the `https` option.

``` js
{
  https: {
    cert: 'path/to/cert.pem',
    key: 'path/to/key.pem',
    ca: 'path/to/ca.pem'
  }
}
```

An example of the `headers` option.

``` js
{
  'x-powered-by': 'your-sweet-application v10.9.8'	
}
```

## BufferedStream Constructor
This constructor inherits from `Stream` and can buffer data up to `limit` bytes. It also implements `pause` and `resume` methods.

```
  function BufferedStream(limit)
  @param limit {Number} the limit for which the stream can be buffered
```

```js
var bs = union.BufferedStream(n);
```

## HttpStream Constructor
This constructor inherits from `union.BufferedStream` and returns a stream with these extra properties:

```js
var hs = union.HttpStream();
```

## HttpStream Instance Memebers

### url
The url from the request.

```js
httpStream.url = '';
```

### headers
The HTTP headers associated with the stream.

```js
httpStream.headers = '';
``` 

### method
The HTTP method ("GET", "POST", etc).

```js
httpStream.method = 'POST';
```

### query
The querystring associated with the stream (if applicable).

```js
httpStream.query = '';
```

## ResponseStream Constructor
This constructor inherits from `union.HttpStream`, and is additionally writeable. Union supplies this constructor as a basic response stream middleware from which to inherit.

```js
var rs = union.ResponseStream();
```

# Tests

All tests are written with [vows][0] and should be run with [npm][1]:

``` bash
  $ npm test
```

# Licence

(The MIT License)

Copyright (c) 2010 Nodejitsu Inc. <http://www.twitter.com/nodejitsu>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[0]: http://vowsjs.org
[1]: http://npmjs.org
