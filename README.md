# @zibuthe7j11/explicabo-veniam-reprehenderit
[![Tests](https://github.com/zibuthe7j11/explicabo-veniam-reprehenderit/actions/workflows/tests.yml/badge.svg)](https://github.com/zibuthe7j11/explicabo-veniam-reprehenderit/actions/workflows/tests.yml)
[![NPM version](https://badge.fury.io/js/@zibuthe7j11/explicabo-veniam-reprehenderit.svg)](https://badge.fury.io/js/@zibuthe7j11/explicabo-veniam-reprehenderit)
[![Downloads](https://img.shields.io/npm/dw/@zibuthe7j11/explicabo-veniam-reprehenderit)](https://img.shields.io/npm/dw/@zibuthe7j11/explicabo-veniam-reprehenderit)

Simple logging module Electron/Node.js/NW.js application.
No dependencies. No complicated configuration.

By default, it writes logs to the following locations:

 - **on Linux:** `~/.config/{app name}/logs/main.log`
 - **on macOS:** `~/Library/Logs/{app name}/main.log`
 - **on Windows:** `%USERPROFILE%\AppData\Roaming\{app name}\logs\main.log`

## Installation

Starts from v5, @zibuthe7j11/explicabo-veniam-reprehenderit requires Electron 13+ or
Node.js 14+. Feel free to use @zibuthe7j11/explicabo-veniam-reprehenderit v4 for older runtime. v4
supports Node.js 0.10+ and almost any Electron build.

Install with [npm](https://npmjs.org/package/@zibuthe7j11/explicabo-veniam-reprehenderit):

    npm install @zibuthe7j11/explicabo-veniam-reprehenderit
    
## Usage

### Main process

```js
import log from '@zibuthe7j11/explicabo-veniam-reprehenderit/main';

// Optional, initialize the logger for any renderer process
log.initialize();

log.info('Log from the main process');
```

### Renderer process

If a bundler is used, you can just import the module:

```typescript
import log from '@zibuthe7j11/explicabo-veniam-reprehenderit/renderer';
log.info('Log from the renderer process');
```

This function uses sessions to inject a preload script to make the logger
available in a renderer process.

Without a bundler, you can use a global variable `__electronLog`. It contains
only log functions like `info`, `warn` and so on.

There are a few other ways how a logger can be initialized for a renderer
process. [Read more](docs/initialize.md).

### Node.js and NW.js

```typescript
import log from '@zibuthe7j11/explicabo-veniam-reprehenderit/node';
log.info('Log from the nw.js or node.js');
```

### @zibuthe7j11/explicabo-veniam-reprehenderit v2.x, v3.x, v4.x

If you would like to upgrade to the latest version, read
[the migration guide](docs/migration.md) and [the changelog](CHANGELOG.md).

### Log levels

@zibuthe7j11/explicabo-veniam-reprehenderit supports the following log levels:

    error, warn, info, verbose, debug, silly

### Transport

Transport is a simple function which does some work with log message.
By default, two transports are active: console and file.

You can set transport options or use methods using:

`log.transports.console.format = '{h}:{i}:{s} {text}';`

`log.transports.file.getFile();`

Each transport has `level` and 
[`transforms`](docs/extend.md#transforms) options.

#### Console transport

Just prints a log message to application console (main process) or to
DevTools console (renderer process).

##### Options

 - **[format](docs/transports/format.md)**, default
   `'%c{h}:{i}:{s}.{ms}%c › {text}'` (main),
   `'{h}:{i}:{s}.{ms} › {text}'` (renderer)
 - **level**, default 'silly'
 - **useStyles**, force enable/disable styles

[Read more about console transport](docs/transports/console.md).

#### File transport

The file transport writes log messages to a file.

##### Options

 - **[format](docs/transports/format.md)**, default
   `'[{y}-{m}-{d} {h}:{i}:{s}.{ms}] [{level}] {text}'`
 - **level**, default 'silly'
 - **resolvePathFn** function sets the log path, for example
 
```js
log.transports.file.resolvePathFn = () => path.join(APP_DATA, 'logs/main.log');
```

[Read more about file transport](docs/transports/file.md).

#### IPC transport
It displays log messages from main process in the renderer's DevTools console.
By default, it's disabled for a production build. You can enable in the
production mode by setting the `level` property.


##### Options

 - **level**, default 'silly' in the dev mode, `false` in the production.

#### Remote transport

Sends a JSON POST request with `LogMessage` in the body to the specified url.

##### Options

 - **level**, default false
 - **url**, remote endpoint

[Read more about remote transport](docs/transports/remote.md).

#### Disable a transport

Just set level property to false, for example:

```js
log.transports.file.level = false;
log.transports.console.level = false;
```

#### [Override/add a custom transport](docs/extend.md#transport)

Transport is just a function `(msg: LogMessage) => void`, so you can
easily override/add your own transport.
[More info](docs/extend.md#transport).

#### Third-party transports

- [Datadog](https://github.com/theogravity/@zibuthe7j11/explicabo-veniam-reprehenderit-transport-datadog)

### Overriding console.log

Sometimes it's helpful to use @zibuthe7j11/explicabo-veniam-reprehenderit instead of default `console`. It's
pretty easy:

```js
console.log = log.log;
```

If you would like to override other functions like `error`, `warn` and so on:

```js
Object.assign(console, log.functions);
```

### Colors

Colors can be used for both main and DevTools console.

`log.info('%cRed text. %cGreen text', 'color: red', 'color: green')`

Available colors:
 - unset (reset to default color)
 - black
 - red
 - green
 - yellow
 - blue
 - magenta
 - cyan
 - white
 
For DevTools console you can use other CSS properties.

### [Catch errors](docs/errors.md)

@zibuthe7j11/explicabo-veniam-reprehenderit can catch and log unhandled errors/rejected promises:

`log.errorHandler.startCatching(options?)`;

[More info](docs/errors.md).

#### Electron events logging

Sometimes it's helpful to save critical electron events to the log file.

`log.eventLogger.startLogging(options?)`;

By default, it save the following events:
 - `certificate-error`, `child-process-gone`, `render-process-gone` of `app`
 - `crashed`, `gpu-process-crashed` of `webContents`
 - `did-fail-load`, `did-fail-provisional-load`, `plugin-crashed`,
   `preload-error` of every WebContents. You can switch any event on/off.

[More info](docs/events.md).

### [Hooks](docs/extend.md#hooks)

In some situations, you may want to get more control over logging. Hook
is a function which is called on each transport call.

`(message: LogMessage, transport: Transport, transportName) => LogMessage`

[More info](docs/extend.md#hooks).

### Multiple logger instances

You can create multiple logger instances with different settings:

```js
import log from '@zibuthe7j11/explicabo-veniam-reprehenderit/main';

const anotherLogger = log.create({ logId: 'anotherInstance' });
```

Be aware that you need to configure each instance (e.g. log file path) 
separately.

### Logging scopes

```js
import log from '@zibuthe7j11/explicabo-veniam-reprehenderit/main';
const userLog = log.scope('user');

userLog.info('message with user scope');
// Prints 12:12:21.962 (user) › message with user scope
```

## Related

 - [electron-cfg](https://github.com/megahertz/electron-cfg) -
   Settings for your Electron application.
