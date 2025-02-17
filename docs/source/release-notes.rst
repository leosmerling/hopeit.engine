Release Notes
=============

Version 0.10.0
______________
- Plugins:
  - This release adds general support in several plugins to properly handle events that are plugged into app endpoints.

  - Apps Client:
    - Support for two authentication strategies: FORWARD_CONTEXT to propagate basic auth from client to server, and
  CLIENT_APP_PUBLIC_KEY to create Bearer token to be validated by server.
    - Added support to configure and call plugin events that are plugged into app endpoints (plug_mode=ON_APP)

  - Config Manager:
    - Returns effective_events section prefixing event names with app_key and plugin_key

  - Apps Visualizer:
    - Handles edges between client apps calling ON_APP plugged events
    - Live stats considers IGNORED events as a warning status
    - Fixed Open API warning for multiple schemas with same name

  - Log Streamer:
    - Support to capture IGNORED (Unathorized) event calls

- Engine:
  - Added tracking in EventContext for app_key and plugin_key, allowing logging those details as extra fields. 


Version 0.9.4
_____________
- Fix: `apps-visualizer` plugin load effective_events from `config-manager` to avoid the need to install monitored apps in the same running environment as `apps-visualizer`
- `config-manager` plugins, exposes effective_events (events with intermediate streams) as part of runtime app info.


Version 0.9.3
_____________
- Fix: pinned `aiohttp_swagger3` version to prevent failure on unsecured endpoints


Version 0.9.2
_____________
- Fix: added missing packages to `apps_visualizer` plugin dist


Version 0.9.1
_____________
- Run single QA pipeline before publishing to PyPi


Version 0.9.0
_____________
- Engine support to configure `AppConnections` and `EventConnections` to express App/Event dependencies.
- Engine support for multiple client implementations via plugins
- App config support for `settings` section in order to enable plugins to use custom schemas to parse configuration values.

- Plugins:
  - Apps Client (new plugin): `hopeit.apps_client` allows invocation of other running apps via http GET or POST requests.
  Enables in a single function call `app_call` to invoke remote app events. See `apps/examples/client-example` for usage scenarios.
  - Apps Visualizer plugin: support for showing connections between connected apps.
  - Basic Auth: tokens are generated using `app_key` from `context`. This means that in order for a token to be accepted
  by a given app, it must be called from the same app. `basic_auth` demo plugin enforces this by making `login` and `refresh`
  endpoints of type `EMBEDDED`, what makes `app_key` from app containing the plugin, to be used when creating the token
  (and not the plugin `app_key`)

- BUG FIXES:
  - Engine: fixed a bug preventing `{...}` expressions in config files pointing to dictionaries to be properly replaced by its value.
  - Security: fixed a bug where sometimes authentication is allowed erroneously when multiple auth methods are configured for a single event.

- BREAKING CHANGES:
  - Engine `auth` module now creates and stores one pair of private/public keys per each running app. Keys are stored
to `.secrets/.private` and `.secrets/public` using `app_key` as a prefix for the file name.
    - All auth tokens from now are validated using the public key of the app creating the token, extracting `app` field from the payload.
    - `new_token` method requires an app_key as a parameter.
    - In order to validate tokens, payload must contain the generating `app_key` in the token payload `app` field.
    - To perform app-to-app authentication, in order to allow an App to be called using `hopeit.apps_client`, the public key of
    the caller app must be accessible in the `.secrets/public` folder of the called application.
    - In production environments, this keys must be mounted/accessible before server starts. It is also recommended to disable automatic
    key generation in server config file.


Version 0.8.3
_____________

- BREAKING CHANGES:
  - class `Json` from `hopeit.dataobjects.jsonify` renamed to `Payload` and moved to `hopeit.dataobjects.payload` for more intuitive usage of @dataobject decorated object. `Json` will be deprecated in a future version.


Version 0.8.2
_____________
- Fix: some management routes to start/stop streams were not working: normalized $ sign to / in route names.


Version 0.8.1
_____________
- `hopeit.dataobjects.jsonify` module: added utility functions to convert dictionaries and list to dataobjects and back


Version 0.8.0
_____________
- Config Manager Plugin: added support to access current process configuration with special hostname "in-process"
- Apps Visualizer plugin:
  - Now can (and should) run separately from the apps/servers that is monitoring
  - Supports connection to remote hosts running config-manager plugin
  - Added list of hosts and status (ALIVE if reachable, ERROR if not)
  - Filter config and live activity by host/group of hosts by name
  - Automatic refresh servers/hosts status
  - Automatic refresh list of active apps
  - Automatic refresh graph on configuration or hosts availability changes


Version 0.7.3
_____________
- Including type information in PIP packages for `hopeit.engine` and plugins.


Version 0.7.2
_____________
- Engine setup: pinned dependencies version when specified in requirements.txt, fallback to requirements.lock when not pinned in txt.
- Apps Visualizer plugin setup: added py.typed marker


Version 0.7.1
_____________
- Config Manager Plugin: Moved cluster_apps_config logic to client that can be used from other apps or plugins.


Version 0.7.0
_____________
- Config Manager Plugin: allows remote access to runtime configuration for `hopeit.engine` servers and clusters


Version 0.6.0
_____________
- Apps Visualizer plugin: supports now live apps activity visualization when used in combination with `log-streamer`
- Apps Visualizer plugin: improved visualization rendering, filters and options.


Version 0.5.0
_____________
- New plugin: `log-streamer` read logs generated by hopeit.engine apps and publish entries to a stream enabling downstream usage like monitoring, dumping log data and analytics.


Version 0.4.3
_____________
- FIX: Missing template on app-visualizer wheel


Version 0.4.2
_____________
- FIX: `date` and `datetime` types are handled according to OpenAPI specs in query string parameters. This is not a breaking change but consider checking that for existing date/datetime query args value format will be validated at request time starting this version.


Version 0.4.1
_____________
- FIX: Missing template on app-visualizer


Version 0.4.0
_____________
- Streams: 
  - Added support for multiple `queues` in `read_stream` and `write_stream` configuration, allowing to produce and consume events in parallel from different sources. hopeit.engine automatically manages independent streams for each queue and ensures a message read from a queue is propagated downstream using the same queue.

- Web: 
  - Support for custom response `content-type` in `PostProcessHook`, i.e. to return `text/plain` or `text/html` for specific applications, instead of default `application/json`.

- Open API:
  - Fixed "Authorization required" with openapi generated entry when endpoint is marked as "Unsecured"

- Plugins: 
  - New plugin for visualizing running configuration (events & streams): `ops/apps-visualizer` plugin.

- BREAKING CHANGES:
  - When an app event is configured with a custom `route` entry to be used instead of app and event name. If it starts with a slash ('/'), route namespace prefix `/api` will be ignored. This can be used to map events to the root endpoint `/` namespace. To ensure default namespace is used, remove starting slash (`/`) from route names.


Version 0.3.0
_____________
- Moved `hopeit.toolkit.storage.redis` to `hopeit.redis-storage` plugin.
- Moved RedisStreamManager to its own plugin. 
- Moved `hopeit.toolkit.storage.fs` to `hopeit.fs-storage` plugin.
- Added test build for Python 3.9

- FIXES: 
  - Removed aiohttp dependency for hopeit.app.context module, in order to allow engine usage on applications that do not require web server module.

- BREAKING CHANGES:
  - By default `stream-manager` is not configured. To enable Redis Streams in server: 1) Install using `pip install hopeit.engine[redis-streams]`, 2) Add `stream_manager=hopeit.redis_streams.RedisStreamManager` to streams section in server config file.
  - Redis Storage toolkit (now a plugin) needs to be installed using `pip install hopeit.redis-storage`
  - Removed `hopeit.dataobjects.validation` and `hopeit.toolkit.validators` modules
  - make simple-example app to match Major. Minor version number from engine. This is only breaking changes for users of this app config file.
  - make simple-benchmark app to match Major. Minor version number from engine. This is only breaking changes for users of this app config file.
  - make basic-auth plugin to match Major. Minor version number from engine. This is only breaking changes for users of this plugin config file.
  

Version 0.2.3
_____________
- Remove unnecessary decode when parsing payload on web module 
- Split generic Stream Manager from Redis specifics, on preparation to support different stream managers
- Made `stream-manager` a configuration option (defaults to same RedisStreamManager used before)


Version 0.2.0
_____________
- MULTIPART uploads http endpoints support: post form-data with file attachments in request (with Json response)
- Support for `__preprocess__` web requests in GET, POST and MULTIPART endpoints
- Ability to define `content-type` for responses with binary files in Open API specification
- DEPRECATION: `title` parameter removed in `app.api.event_api(...)` in favor of `summary` and `description`


Version 0.1.5
_____________
- Automatic publishing to PyPi
- Open API: added summary and description parameters to Open API specification. Deprecation warning for title param.
- FIX: Improved dependency handling
- FIX: fix object listing in FileStorage toolkit


Version 0.1.0
_____________

Initial __hopeit.engine__ version support for:
- Enables development of microservices in Python (3.7+)
- Provides aiohttp web server for API endpoints.
- Open API schema validation and docs site.
- Modular and testable application design: each microservice is an app composed of independent events
- Logging of event invocations and results.
- Metrics: event durations, events starts, success, failures. Stream processing rates.
- Tracking/tracing: keep track of request ids among applications and multiple events execution.
- Event publishing and consuming to Redis Streams.
- Engine core support for functional Events with Steps
- Multiple microservices definition as Apps
- GET, POST http endpoints with JSON responses
- STREAM events to asynchroously consume and process messages
- SERVICE events for continuously running processes
- read_stream / write_stream support for Redis streams
- OpenAPI specification support for HTTP endpoints
- Dataobjects with Json Schema validation
- JSON configuration files with Json Schema validation
- Collector steps pattern support for concurrent execution of steps using asyncio
- hopeit_server command line interface
- hopeit_openapi command line interface
- Helps to create elegant and well structure code using your preferred IDE.
- Data Science / Machine Learning friendly: applications can be developed and tested using Jupyter Notebooks.
- Testing: provides utilities to test from Notebooks or Python testing frameworks.
