StandardHttpError.js
====================
[![NPM version][npm-badge]](http://badge.fury.io/js/standard-http-error)
[npm-badge]: https://badge.fury.io/js/standard-http-error.png

**StandardHttpError.js** is a very simple but useful **Error** class for Node.js
that represents HTTP errors. You can then detect it with `instanceof` in error
handling middleware and act accordingly.

You can use StandardHttpError.js with any error code you like, standardized or
not. They don't have to exist beforehand, so if you're living on the cutting
edge, feel free to use `new HttpError(451, "Unavailable For Legal Reasons")`.


Installing
----------
```sh
npm install standard-http-error
```

StandardHttpError.js follows [semantic versioning](http://semver.org/), so feel
free to depend on its major version with something like `>= 1.0.0 < 2`
(a.k.a `^1.0.0`).


Using
-----
```javascript
var HttpError = require("standard-http-error")
throw new HttpError(404)
```

Your error handler will then receive an instance of `HttpError` along with the
following enumerable properties:

Property | Value
---------|------
name     | `"HttpError"`
code     | `404`
message  | `"Not Found"`

As always for errors, the non-enumerable `stack` property is there as well.

For compatibility with Express.js's default request handler
([finalhandler](https://www.npmjs.com/package/finalhandler) — the one that
prints your errors out if you don't handle them), StandardHttpError.js also sets
`status`, `statusCode` and `statusMessage` to be aliases of `code` and
`message`. They're non-enumerable to not pollute serialization.

### Creating a new instance by error name
StandardHttpError.js also supports passing a constant name instead of the error
code. Constant names are taken from Node.js's `Http` module's `STATUS_CODES`
object so they'll always be up to date.

```javascript
new HttpError("NOT_FOUND")
new HttpError("FORBIDDEN")
```

See below for a [list of error code names](#error-codes).

### Setting a custom message
```javascript
new HttpError(412, "Bad CSRF Token")
```

The default "Precondition Failed" message that the error code 412 would've
resulted in will then be replaced by "Bad CSRF Token".

Note that status messages were always meant to be human readable, so it's
perfect fine and even preferable to provide clarification in the status message.
Try to stick to the capitalized form, though, as that will match the default
HTTP status message style.

### Setting custom properties
You can pass custom properties to be attached to the error instance as an
object:

```javascript
new HttpError(404, {url: req.url})
new HttpError(412, "Bad CSRF Token", {session: req.session})
```

You can access the given `session` property then as `err.session`.


### Subclassing StandardHttpError
If you wish to add your own functionality to StandardHttpError, subclass it as:

```javascript
var HttpError = require("standard-http-error")

function RemoteError(res) {
  HttpError.call(res.statusCode, res.statusMessage)
}

RemoteError.prototype = Object.create(HttpError.prototype, {
  constructor: {value: RemoteError, configurable: true, writeable: true}
})
```

The [StandardError.js](https://github.com/moll/js-standard-error) library that
StandardHttpError.js uses makes sure the `name` and `stack` properties of your
new error class are set properly.

If you don't want your new error class to directly inherit from
`StandardHttpError`, feel free to leave the `RemoteError.prototype` line out.
Everything will work as before except your `RemoteError` will no longer be an
`instanceof` StandardHttpError.js.  You might want to manually grab the
`HttpError.prototype.toString` function then though, as that's useful for nice
`String(err)` output.

### Switching based on error codes
```javascript
switch (err.code) {
  case HttpError.UNAUTHORIZED: return void res.redirect("/signin")
  case HttpError.NOT_FOUND: return void res.render("404")
  case 451: return void res.redirect("/legal")
  default: return void res.render("500")
}
```

### Middleware
StandardHttpError.js comes very handy when used with Connect.js/Express.js's
error handling functionality:

```javascript
var HttpError = require("standard-http-error")

app.use(function(err, req, res, next) {
  if (!(err instanceof HttpError)) return void next(err)

  res.statusCode = err.code
  res.statusMessage = err.message
  res.render("error", {title: err.message})
})
```


<a name="error-codes" />
Error Codes
-----------
StandardHttpError.js comes with a list of status message constants that you can
use for comparison and in `switch` statements.

```javascript
HttpError.NOT_FOUND // => 404
```

The names are generated automatically from Node.js's `Http.STATUS_CODES` object
during run-time, so they'll always be up to date. Here's the list for reference
as of Node v0.11:

Code  | Name
------|-----
`100` | `CONTINUE`
`101` | `SWITCHING_PROTOCOLS`
`102` | `PROCESSING`
`200` | `OK`
`201` | `CREATED`
`202` | `ACCEPTED`
`203` | `NON_AUTHORITATIVE_INFORMATION`
`204` | `NO_CONTENT`
`205` | `RESET_CONTENT`
`206` | `PARTIAL_CONTENT`
`207` | `MULTI_STATUS`
`300` | `MULTIPLE_CHOICES`
`301` | `MOVED_PERMANENTLY`
`302` | `MOVED_TEMPORARILY`
`303` | `SEE_OTHER`
`304` | `NOT_MODIFIED`
`305` | `USE_PROXY`
`307` | `TEMPORARY_REDIRECT`
`308` | `PERMANENT_REDIRECT`
`400` | `BAD_REQUEST`
`401` | `UNAUTHORIZED`
`402` | `PAYMENT_REQUIRED`
`403` | `FORBIDDEN`
`404` | `NOT_FOUND`
`405` | `METHOD_NOT_ALLOWED`
`406` | `NOT_ACCEPTABLE`
`407` | `PROXY_AUTHENTICATION_REQUIRED`
`408` | `REQUEST_TIME_OUT`
`409` | `CONFLICT`
`410` | `GONE`
`411` | `LENGTH_REQUIRED`
`412` | `PRECONDITION_FAILED`
`413` | `REQUEST_ENTITY_TOO_LARGE`
`414` | `REQUEST_URI_TOO_LARGE`
`415` | `UNSUPPORTED_MEDIA_TYPE`
`416` | `REQUESTED_RANGE_NOT_SATISFIABLE`
`417` | `EXPECTATION_FAILED`
`418` | `IM_A_TEAPOT`
`422` | `UNPROCESSABLE_ENTITY`
`423` | `LOCKED`
`424` | `FAILED_DEPENDENCY`
`425` | `UNORDERED_COLLECTION`
`426` | `UPGRADE_REQUIRED`
`428` | `PRECONDITION_REQUIRED`
`429` | `TOO_MANY_REQUESTS`
`431` | `REQUEST_HEADER_FIELDS_TOO_LARGE`
`500` | `INTERNAL_SERVER_ERROR`
`501` | `NOT_IMPLEMENTED`
`502` | `BAD_GATEWAY`
`503` | `SERVICE_UNAVAILABLE`
`504` | `GATEWAY_TIME_OUT`
`505` | `HTTP_VERSION_NOT_SUPPORTED`
`506` | `VARIANT_ALSO_NEGOTIATES`
`507` | `INSUFFICIENT_STORAGE`
`509` | `BANDWIDTH_LIMIT_EXCEEDED`
`510` | `NOT_EXTENDED`
`511` | `NETWORK_AUTHENTICATION_REQUIRED`


License
-------
StandardHttpError.js is released under a *Lesser GNU Affero General Public
License*, which in summary means:

- You **can** use this program for **no cost**.
- You **can** use this program for **both personal and commercial reasons**.
- You **do not have to share your own program's code** which uses this program.
- You **have to share modifications** (e.g. bug-fixes) you've made to this
  program.

For more convoluted language, see the `LICENSE` file.


About
-----
**[Andri Möll][moll]** typed this and the code.  
[Monday Calendar][monday] supported the engineering work.

If you find StandardHttpError.js needs improving, please don't hesitate to type
to me now at [andri@dot.ee][email] or [create an issue online][issues].

[email]: mailto:andri@dot.ee
[issues]: https://github.com/moll/node-standard-http-error/issues
[moll]: http://themoll.com
[monday]: https://mondayapp.com
