JSON-RPC 2.0 Client for HTTP and WebSocket backends
===================================================

This plugin requires jquery.json.js to be available, or at least the methods $.toJSON and
$.parseJSON.

The plan is to make use of websockets if they are available, but work just as well with only
http if not.

Usage example:

```Javascript
var foo = new $.JsonRpcClient({ ajaxUrl: '/backend/jsonrpc' });
foo.call(
  'bar', [ 'A parameter', 'B parameter' ],
  function(result) { alert('Foo bar answered: ' + result.my_answer); },
  function(error)  { console.log('There was an error', error); }
);
```

The `call` method will return the [deferred object](https://api.jquery.com/category/deferred-object/) that $.ajax returns, or null if websockets are used.

Batch calls
-----------

In HTTP you can batch calls with the batch-method.  You get a batch handler to make all call- and
notify-requests on, and they will all be sent in a single request.  When a WebSocket backend is
available, the requests will be sent immediately.

Example:

```Javascript
var foo = new $.JsonRpcClient({ ajaxUrl: '/backend/jsonrpc' });
foo.batch(
  function(batch) {
    batch.call('bar', [ 'A parameter', 'B parameter' ], success_cb1, error_cb1);
    batch.call('baz', { parameters: 'could be object' }, success_cb2, error_cb2);
  },
  function(all_result_array) { alert('All done.'); },
  function(error_data)       { alert('Error in batch response.'); }
);
```
Each result will be paired with it's own callback.  The all_done_callback given first to batch is
called when all other callbacks are done.


$.JsonRpcClient Options
-----------------------

`ajaxUrl` **string** The URL to use when making requests over HTTP.

`headers` **object** HTTP headers to be passed to $.ajax when making requests over HTTP.

`socketUrl` **string** The URL to use when making requests over WSS (web sockets). Not used if a custom `getSocket` is supplied.

`onmessage` **function** Optional onmessage-handler for WebSocket.

`onopen` **function** Optional onopen-handler for WebSocket.

`onclose` **function** Optional onclose-handler for WebSocket.

`onerror` **function** Optional onerror-handler for WebSocket.

`getSocket` **function** Custom socket supplier for using an already existing socket.


WebSocket
---------

If a websocket backend is given, it will be used if the browser supports it:

```Javascript
var foo = new $.JsonRpcClient({ ajaxUrl: '/backend/jsonrpc', socketUrl: 'ws://example.com/' });
foo.call('bar', [ 'param' ], success_cb, error_cb);
--> websocket message: {"jsonrpc":"2.0","method":"bar","params":["param"],"id":3}
```

The http fallback will be used when the browser is not WebSocket capable, but NOT when the
websocket fails to connect.


### WebSocket other message handler

If a non-response message comes in, it can be forwarded to an external handler by giving the
onmessage-option.


### Using an already alive websocket - getSocket option

If you already have a websocket active and want that to be used for the JSON-RPC requests, you can
use the getSocket option.  getSocket should point to a function with the following interface:
```
@param onmessage_cb  getSocket will be called with an onmessage_cb that must be bound to the
                     onmessage event of the returned socket.

@return websocket|null  The returned object should act like a WebSocket: it must have the
                        property readyState, with a value of less than or equal to 1.  If less
                        than 1, it must have an onopen-property that can be set, and that will
                        be called when the socket is ready.  Also, it must be have the function
                        'call', taking a string.
                        It could also return null if no socket is available.
```

The main purpose of this is to couple the client with a matching server, that can take requests
from the backend.


Test
----

The test-file is supposed to be run with [JsTestDriver](https://code.google.com/p/js-test-driver/).

To run the tests from the command line, use the following commands:

    # Step 1 - Start jsTestDriver server on port 9000 on localhost
    java -jar /path/to/jsTestDriver.jar --port 9000

    # Step 2 - Browse to http://localhost:9000/capture with one or more browsers
    # Each browser you direct to that address will run the tests.

    # Step 3 - Run tests
    java -jar /path/to/jsTestDriver.jar --reset --config jsTestDriver.conf --tests all

JSON-RPC 2.0
------------

JSON-RPC 2.0 is a very simple protocol for remote procedure calls, agnostic of carrier (http,
websocket, tcp, whatever…).

[JSON-RPC 2.0 Specification](http://www.jsonrpc.org/specification)
