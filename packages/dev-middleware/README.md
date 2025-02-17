# @react-native/dev-middleware

![npm package](https://img.shields.io/npm/v/@react-native/dev-middleware?color=brightgreen&label=npm%20package)

Dev server middleware supporting core React Native development features. This package is preconfigured in all React Native projects.

## Usage

Middleware can be attached to a dev server (e.g. [Metro](https://facebook.github.io/metro/docs/getting-started)) using the `createDevMiddleware` API.

```js
import { createDevMiddleware } from '@react-native/dev-middleware';

function myDevServerImpl(args) {
  ...

  const {middleware, websocketEndpoints} = createDevMiddleware({
    host: args.host,
    port: metroConfig.server.port,
    projectRoot: metroConfig.projectRoot,
    logger,
  });

  await Metro.runServer(metroConfig, {
    host: args.host,
    ...,
    unstable_extraMiddleware: [
      middleware,
      // Optionally extend with additional HTTP middleware
    ],
    websocketEndpoints: {
      ...websocketEndpoints,
      // Optionally extend with additional WebSocket endpoints
    },
  });
}
```

## Included middleware

`@react-native/dev-middleware` is designed for integrators such as [`@expo/dev-server`](https://www.npmjs.com/package/@expo/dev-server) and [`@react-native/community-cli-plugin`](https://github.com/facebook/react-native/tree/main/packages/community-cli-plugin). It provides a common default implementation for core React Native dev server responsibilities.

We intend to keep this to a narrow set of functionality, based around:

- **Debugging** — The [Chrome DevTools protocol (CDP)](https://chromedevtools.github.io/devtools-protocol/) endpoints supported by React Native, including the Inspector Proxy, which facilitates connections with multiple devices.
- **Dev actions** — Endpoints implementing core [Dev Menu](https://reactnative.dev/docs/debugging#accessing-the-dev-menu) actions, e.g. reloading the app, opening the debugger frontend.

### HTTP endpoints

<small>`DevMiddlewareAPI.middleware`</small>

These are exposed as a [`connect`](https://www.npmjs.com/package/connect) middleware handler, assignable to `Metro.runServer` or other compatible HTTP servers.

#### GET `/json/list`, `/json` ([CDP](https://chromedevtools.github.io/devtools-protocol/#endpoints))

Returns the list of available WebSocket targets for all connected React Native app sessions.

#### GET `/json/version` ([CDP](https://chromedevtools.github.io/devtools-protocol/#endpoints))

Returns version metadata used by Chrome DevTools.

#### POST `/open-debugger`

Open the JavaScript debugger for a given CDP target (direct Hermes debugging).

<details>
<summary>Example</summary>

    curl -X POST 'http://localhost:8081/open-debugger?appId=com.meta.RNTester'
</details>

### WebSocket endpoints

<small>`DevMiddlewareAPI.websocketEndpoints`</small>

#### `/inspector/device`

WebSocket handler for registering device connections.

#### `/inspector/debug`

WebSocket handler that proxies CDP messages to/from the corresponding device.
