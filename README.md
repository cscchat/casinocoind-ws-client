# casinocoind-ws-client

#### A lightweight reconnecting (health checking) websocket client for `casinocoind`

This is a websocket client for [casinocoind](https://casinocoin.org/build/reference-casinocoind.html).
Of course there's [casinocoin-libjs](https://github.com/casinocoin/casinocoin-libjs), but the uncompressed browserified version
of casinocoin-libjs is too big for mobile users.

> But you can compress, gzip, minify, etc.

That's right, but since we're in the business of crypto, we might want to offer our users the entire uncompressed source, so they can audit it.

> But casinocoin-libjs comes with all the signing goodies.

That's right. But you can build amazing things without signing anything. **If you want to sign offline, or online and await the transaction state, check out the complementary sign repo: [casinocoind-ws-client-sign](https://www.npmjs.com/package/casinocoind-ws-client-sign)**.

> But casinocoin-libjs has helper methods

Yeah, well... Sometimes the helper methods are not that great. The [Casinocoin docs](https://casinocoin.org/build/reference-casinocoind.html) show you all the plain JSON commands. I'd rather send those, so the client is 1:1 compatible with the docs.

## So. You want to use **casinocoind-ws-client**?

Great 🎉! A few notes:

- `casinocoind-ws-client` is _promise_-based. You'll need a [polyfill](https://cdn.jsdelivr.net/npm/promise-polyfill/dist/polyfill.min.js) for IE10 and other crappy browsers.
- You can use `casinocoind-ws-client` in the browser and in nodejs 😎
- `casinocoind-ws-client` will auto-reconnect. But: **not only when the WebSocket disconnects**! The client will send `ping` requests to the casinocoind-server every few seconds. If the client (four times in a row) doesn't receive a response, the WebSocket will disconnect. Either the client or the server is offline.
- `casinocoind-ws-client` will _always_ subscribe on _ledger_-events. This way the `getState()` method can always tell you the most recent ledger.
- `casinocoind-ws-client` will emit events.
- If you want to **test the reconnecting and health checking** (e.g.: connected but no response from the casinocoind server = auto-reconnect) you can use **[this gist](https://gist.github.com/cscchat/2f21a887c6e1e7448e3fa55e75801b89)** to setup a casinocoind websocket proxy. You can start / quit / silence any time by restarting the script 🍻 (_The script is configured to stop passing messages from casinocoind to the client 20 seconds after connecting_).

### Use **casinocoind-ws-client** in nodejs

First install the module;
```
npm install --save casinocoind-ws-client
```

Now require:

```
const casinocoindWsClient = require('casinocoind-ws-client')
```

... and use:

```
new casinocoindWsClient('wss://...').then(...).catch(...)
```

Samples are available in [the Github repo](https://github.com/cscchat/casinocoind-ws-client/tree/master/samples). There are samples on signing and submitting transactions [over here, in the casinocoind-ws-client-sign repo](https://github.com/cscchat/casinocoind-ws-client-sign/blob/master/samples).

### Use **casinocoind-ws-client** in vanillajs

The client is written in ES6 (e.g. [arrow functions](https://hacks.mozilla.org/2015/06/es6-in-depth-arrow-functions/)). The browser won't understand all the code.

Thank goodness you can use [browserify](http://browserify.org/) to convert the code.

1. Checkout the [Github-repo](https://github.com/cscchat/casinocoind-ws-client)
2. Enter the directory of your checkout using your commandline (make sure you have [nodejs](https://nodejs.org/en/download/) installed)
3. Install the dependencies; `npm install`
4. Install browserify; `npm install -g browserify` (the `-g` will install globally)
5. Browserify the code:

```
browserify -r .:casinocoind-ws-client -o dist/casinocoind-ws-client.js
```

This will convert/compile the ES6 code to a browser-compatible .js file, stored in the `dist/` folder.

Here's a [sample jsfiddle](https://jsfiddle.net/cscchat/52muey62/33/) with a precompiled version 😎

### Use **casinocoind-ws-client** in vue-webpack

Just `npm install --save casinocoind-ws-client` in your vue-webpack project. You can now start your module with:

```
import casinocoindWsClient from './npm-module/casinocoind-ws-client.js'
```

... and kick off the constructor somewhere;

```
new casinocoindWsClient('wss://...').then(...).catch(...)
```

If vue-webpack starts bugging you with an error like this: `Cannot assign to read only property 'exports' of object '#<Object>'`, remove the `transform-runtime` plugin from `.babelrc`.

# Docs

The docs will soon move to a dedicated website. For now (since we're in the beta phase anyway) the samples below will have to do.

> The code below is written using vanillajs code. I'd prefer to use arrow functions, but to make the docs compatible with plain old javascript, I decided to do it the old fashioned way.

## 1. Connecting

That's easy. You construct a new `casinocoindWsClient` to the WebSocket-server. Please note: use `ws://` for **http** connections, and `wss://` for **https** connection. Actually: don't use http. Use https. Always. Except if you are developing on your own machine.

```
new casinocoindWsClient('wss://ws01.casinocoin.org:4443').then(function (connection) {
  // We have liftoff!
  // All or other code lives here, using the 'connection' variable
}).catch(function (error) {
  // Oops!
})
```

You should never reach the `catch`. Not even when the connection drops. The client will auto-reconnect. You'll only reach the `catch` if something goes wrong initially, e.g. the URI you are connecting to is invalid.

### Reconnecting

When the connection can't be setup, the connection drops, the internet connection times out, the casinocoind server goes offline, whatever: the client will reconnect at increasing intervals (min: 2 sec., max: 60 sec.). When the connection is online again, all the [subscriptions](https://casinocoin.org/build/reference-casinocoind.html/#subscribe) you sent will be re-sent.

If your initial connection is ready, the (promise) `then` will execute. If you had a working connection before, you lost the connection and the client reconnected, an event will be emitted (see _2. Events_).

### State

If you have / had a connection, in the (promise) `then` the method `getState` is available. Assuming your then-callback variable is called `connection`:

```
console.log(connection.getState())
```

... will show you the online/offline state, fee, latency (ms), most recent ledger, etc. Sample response:

```
{ online: true,
  latencyMs: { last: 536, avg: 536, secAgo: 3.338 },
  server:
   { version: '0.90.0',
     publicKey: 'c9KcmEKTW3ggFgTjNMVkJwJ5R8RhQZeacYLTVgWFcnwheniS7zGA',
     uri: 'wss://ws01.casinocoin.org' },
  ledger: { last: 37064724, validated: '32570-37064724', count: 37032154 },
  fee: { last: 17.578125, avg: 17.578125, secAgo: 3.339 },
  secLastContact: 0.001 }
```

### Disconnecting

To disconnect:

```
connection.close().then(function (closeInfo) {
 console.log('Closed', closeInfo)
}).catch(function (error) {
 console.log('Close error', error)
})
```

Once closed no reconnects will be attempted. The `catch` will only fire if the connection is already closed.

## 2. Events

The `casinocoind-ws-client` will emit the events below. You can subscribe to events this way:

```
connection.on('ledger', function (ledger) {
  console.log('Ledger Closed:', ledger)
})
```

- **`error`**
An error occurred (See _4. Errors_)
- **`retry`**
The could not be setup. A reconnect attempt will take place.
- **`close`**
The connection closed
- **`reconnect`**
The connection is lost or assumed lost. A reconnect will take place.
- **`state`**
The connection state changed (online (_true_) / offline (_false_))
- **`ledger`**
A ledger closed
- **`transaction`**
A new transaction event (subscription required, see _3A. Subscribe_)
- **`validation`**
- A new validation event (subscription required, see _3A. Subscribe_)

## 3. Methods (sending commands)

These methods are available on the `connection` (promise `then` callback argument):

- **`on`**, see _2. Events_
- **`getState`**, see _State_
- **`close`**, see _Disconnecting_
- **`send`**: send a command, and return a promise (with the reply from **casinocoind**)

Any (non-admin) command from the [casinocoind WebSocket docs](https://casinocoin.org/build/casinocoin-api-tool.html) can be sent.

Here's a [sample jsfiddle](https://jsfiddle.net/cscchat/52muey62/33/).
Below are two examples:

#### Example 1: getting [server_info](https://casinocoin.org/build/casinocoin-api-tool.html/#server-info): `server_info`
```
connection.send({
  command: 'server_info'
}).then(function (info) {
  console.log('Got server info:', info)
}).catch(function (error) {
  console.log('Got error', error)
})
```

#### Example 2: getting [transaction](https://casinocoin.org/build/casinocoin-api-tool.html/#tx) details: `tx`

```
connection.send({
  command: 'tx',
  transaction: '2ABE23DC1E7C91E2E47C367833E4D1BEF58D43D1E5CBFFA6107F97FD8978B161'
}).then(function (info) {
  console.log('Got server info:', info)
}).catch(function (error) {
  console.log('Got error', error)
})
```

## 3A. Subscribe (events)

If you want the `casinocoind` server to send you events, you have to [subscribe](https://casinocoin.org/build/casinocoin-api-tool.html/#subscribe). After subscribing to events, the `casinocoind-ws-client` will emit events when they come in (like: `transaction` or `validation`, see _2. Events_)

You can subscribe to **accounts** or **streams**:

```
connection.send({
  command: 'subscribe',
  streams: [ 'validations', 'transactions_proposed' ]
}).then(function (response) {
  console.log('Subscribed', response)
}).catch(function (error) {
  console.log('Subscribe error', error)
})
```

And / or you can subscribe to all transaction events for one or more wallets:

```
connection.send({
  command: 'subscribe',
  accounts: [ 'cDarPNJEpCnpBZSfmcquydockkePkjPGA2', 'caSnj7PCzBm5ZxC8Kbs1pBfouTLZnfjVja', 'cJ5baKjE8WknnaFCEu1tt2rQb2rAAbZ1Wo' ]
}).then(function (response) {
  console.log('Subscribed', response)
}).catch(function (error) {
  console.log('Subscribe error', error)
})
```

You can always add more subscriptions or `unsubscribe`.

> Please note: if you unsubscribe the `ledger`-stream, the class may reconnect!

## 4. Errors

When shit doesn't hit the fan but something goes wrong, an `error` event will be emitted. An `error` event will contain at least an `type` and `error` attribute. The following `type` values exist:

- **`ping_hiccup`** if no ping/pong response was received from the server after a few seconds. Maybe we are offline, or the server is offline or unresponsive? After 4 hiccups in a row the client will reconnect.
- **`ping_error`** if a ping resulted in an (WebSocket) error
- **`ping_timeout`** if a ping was sent, but timed out
- **`serverinfo_timeout`** if we requested server_info (after setting up the connection) but we didn't get a response
- **`message_invalid_json`** if we received a message from the server, but it wasn't JSON (should never happen, except if you are not connecting to a casinocoind WebSocket)
- **`message_invalid_response`** if we received a message from the server, but it didn't contain a message id nor was it an event (should never happen, except if you are not connecting to a casinocoind WebSocket)
