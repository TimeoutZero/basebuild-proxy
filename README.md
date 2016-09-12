# basebuild-proxy-rules
===

`basebuild-proxy-rules` is an add-on module to the [node-http-proxy](https://github.com/nodejitsu/node-http-proxy) library. It lets you define a set of rules to translate matching routes to target routes that the reverse proxy service will talk to on the client's behalf.
> Based on [http-proxy-rules](https://github.com/donasaur/http-proxy-rules)

## Installation
```sh
npm install basebuild-proxy-rules --save
```

## Example Use Case
```js
  var http = require('http'),
      httpProxy = require('http-proxy'),
      HttpProxyRules = require('basebuild-proxy-rules');

  // Set up proxy rules instance
  var proxyRules = new HttpProxyRules({
    rules: {
      '.*/test'  : 'http://localhost:8080/cool', // Rule (1)
      '.*/test2/': 'http://localhost:8080/cool2/' // Rule (2)
    },
    default: 'http://localhost:8080' // default target
  });

  // Create reverse proxy instance
  var proxy = httpProxy.createProxy();

  // Create http server that leverages reverse proxy instance
  // and proxy rules to proxy requests to different targets
  http.createServer(function(req, res) {

    // a match method is exposed on the proxy rules instance
    // to test a request to see if it matches against one of the specified rules
    var target = proxyRules.match(req);
    if (target) {
      return proxy.web(req, res, {
        target: target
      });
    }

    res.writeHead(500, { 'Content-Type': 'text/plain' });
    res.end('The request url and path did not match any of the listed rules!');
  }).listen(6010, cb);
```

Given the object we used to initialize the `basebuild-proxy-rules` instance above, here are some [**examples**](test/index.tests.js#L38) of how sample url paths would be translated.

## Options

You can initialize a new instance with the following options:

```js
{
  rules: {

    '/prefix1' : { // Rule options
      removePrefix: true
      target: 'http://localhost:9000'
    },

    '/prefix2' : 'http://localhost:9001',
  },

  // (optional) if no rules matched, translate url path to specified default
  default: 'http://localhost:8080'
}

```
|Option|Type|Description|
|---|---|---|---|
rules| `String|Object` | Route mapping of reverse proxy based on `stringPrefix: stringTarget`  or `stringPrefix: {//options}`
default| `String` | Default target URL

### Rule Options
|Option|Type|Description|Default|
|---|---|---|---|
target| 'String' | Target URL
removePrefix | `Boolean` | A flag to remove the key prefix when it resolves the proxy | false


The rules object contains a set of key-value pairs mapping a regex-supported url path to a target route. The module only tries to match the visited url path, and not the entire url, with a specified rule. The target route must include the protocol (e.g., http) and the FQDN. See the [tests](test/index.tests.js) for examples of how incoming route url paths may be translated with the use of this module.

## Other Notes
* `(?:\\W|$)` is appended to the end of the regex-supported url path, so that if there is a key like  `.*/test` in the rules, the module matches paths `/test`, `/test/`, `/test?` but not `/testing`.
* As long as object keys continued to be ordered in V8, if there are multiple rules that match against a given url path, the module will pick the matching rule listed first for the translation.

## Credit
> [donasaur](https://github.com/donasaur) from [http-proxy-rules](https://github.com/donasaur/http-proxy-rules)

[![Carlos Henrique](https://avatars0.githubusercontent.com/u/2482989?v=3&s=96)](https://github.com/carloshpds) | [![Renato Massi](https://avatars3.githubusercontent.com/u/14281128?v=3&s=96)](https://github.com/RPereiraRedspark)
--- | --- | --- | --- | --- | --- | ---
[Carlos Henrique](https://github.com/carloshpds)| [Renato Massi](https://github.com/RPereiraRedspark)
