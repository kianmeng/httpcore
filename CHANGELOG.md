# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## 0.13.7 (September 13th, 2021)

- Fix broken error messaging when URL scheme is missing, or a non HTTP(S) scheme is used. (Pull #403)

## 0.13.6 (June 15th, 2021)

### Fixed

- Close sockets when read or write timeouts occur. (Pull #365)

## 0.13.5 (June 14th, 2021)

### Fixed

- Resolved niggles with AnyIO EOF behaviours. (Pull #358, #362)

## 0.13.4 (June 9th, 2021)

### Added

- Improved error messaging when URL scheme is missing, or a non HTTP(S) scheme is used. (Pull #354)

### Fixed

- Switched to `anyio` as the default backend implementation when running with `asyncio`. Resolves some awkward [TLS timeout issues](https://github.com/encode/httpx/discussions/1511).

## 0.13.3 (May 6th, 2021)

### Added

- Support HTTP/2 prior knowledge, using `httpcore.SyncConnectionPool(http1=False)`. (Pull #333)

### Fixed

- Handle cases where environment does not provide `select.poll` support. (Pull #331)

## 0.13.2 (April 29th, 2021)

### Added

- Improve error message for specific case of `RemoteProtocolError` where server disconnects without sending a response. (Pull #313)

## 0.13.1 (April 28th, 2021)

### Fixed

- More resiliant testing for closed connections. (Pull #311)
- Don't raise exceptions on ungraceful connection closes. (Pull #310)

## 0.13.0 (April 21st, 2021)

The 0.13 release updates the core API in order to match the HTTPX Transport API,
introduced in HTTPX 0.18 onwards.

An example of making requests with the new interface is:

```python
with httpcore.SyncConnectionPool() as http:
    status_code, headers, stream, extensions = http.handle_request(
        method=b'GET',
        url=(b'https', b'example.org', 443, b'/'),
        headers=[(b'host', b'example.org'), (b'user-agent', b'httpcore')]
        stream=httpcore.ByteStream(b''),
        extensions={}
    )
    body = stream.read()
    print(status_code, body)
```

### Changed

- The `.request()` method is now `handle_request()`. (Pull #296)
- The `.arequest()` method is now `.handle_async_request()`. (Pull #296)
- The `headers` argument is no longer optional. (Pull #296)
- The `stream` argument is no longer optional. (Pull #296)
- The `ext` argument is now named `extensions`, and is no longer optional. (Pull #296)
- The `"reason"` extension keyword is now named `"reason_phrase"`. (Pull #296)
- The `"reason_phrase"` and `"http_version"` extensions now use byte strings for their values. (Pull #296)
- The `httpcore.PlainByteStream()` class becomes `httpcore.ByteStream()`. (Pull #296)

### Added

- Streams now support a `.read()` interface. (Pull #296)

### Fixed

- Task cancelation no longer leaks connections from the connection pool. (Pull #305)

## 0.12.3 (December 7th, 2020)

### Fixed

- Abort SSL connections on close rather than waiting for remote EOF when using `asyncio`.  (Pull #167)
- Fix exception raised in case of connect timeouts when using the `anyio` backend. (Pull #236)
- Fix `Host` header precedence for `:authority` in HTTP/2. (Pull #241, #243)
- Handle extra edge case when detecting for socket readability when using `asyncio`. (Pull #242, #244)
- Fix `asyncio` SSL warning when using proxy tunneling. (Pull #249)

## 0.12.2 (November 20th, 2020)

### Fixed

- Properly wrap connect errors on the asyncio backend. (Pull #235)
- Fix `ImportError` occurring on Python 3.9 when using the HTTP/1.1 sync client in a multithreaded context. (Pull #237)

## 0.12.1 (November 7th, 2020)

### Added

- Add connect retries. (Pull #221)

### Fixed

- Tweak detection of dropped connections, resolving an issue with open files limits on Linux. (Pull #185)
- Avoid leaking connections when establishing an HTTP tunnel to a proxy has failed. (Pull #223)
- Properly wrap OS errors when using `trio`. (Pull #225)

## 0.12.0 (October 6th, 2020)

### Changed

- HTTP header casing is now preserved, rather than always sent in lowercase. (#216 and python-hyper/h11#104)

### Added

- Add Python 3.9 to officially supported versions.

### Fixed

- Gracefully handle a stdlib asyncio bug when a connection is closed while it is in a paused-for-reading state. (#201)

## 0.11.1 (September 28nd, 2020)

### Fixed

- Add await to async semaphore release() coroutine (#197)
- Drop incorrect curio classifier (#192)

## 0.11.0 (September 22nd, 2020)

The Transport API with 0.11.0 has a couple of significant changes.

Firstly we've moved changed the request interface in order to allow extensions, which will later enable us to support features
such as trailing headers, HTTP/2 server push, and CONNECT/Upgrade connections.

The interface changes from:

```python
def request(method, url, headers, stream, timeout):
    return (http_version, status_code, reason, headers, stream)
```

To instead including an optional dictionary of extensions on the request and response:

```python
def request(method, url, headers, stream, ext):
    return (status_code, headers, stream, ext)
```

Having an open-ended extensions point will allow us to add later support for various optional features, that wouldn't otherwise be supported without these API changes.

In particular:

* Trailing headers support.
* HTTP/2 Server Push
* sendfile.
* Exposing raw connection on CONNECT, Upgrade, HTTP/2 bi-di streaming.
* Exposing debug information out of the API, including template name, template context.

Currently extensions are limited to:

* request: `timeout` - Optional. Timeout dictionary.
* response: `http_version` - Optional. Include the HTTP version used on the response.
* response: `reason` - Optional. Include the reason phrase used on the response. Only valid with HTTP/1.*.

See https://github.com/encode/httpx/issues/1274#issuecomment-694884553 for the history behind this.

Secondly, the async version of `request` is now namespaced as `arequest`.

This allows concrete transports to support both sync and async implementations on the same class.

### Added

- Add curio support. (Pull #168)
- Add anyio support, with `backend="anyio"`. (Pull #169)

### Changed

- Update the Transport API to use 'ext' for optional extensions. (Pull #190)
- Update the Transport API to use `.request` and `.arequest` so implementations can support both sync and async. (Pull #189)

## 0.10.2 (August 20th, 2020)

### Added

- Added Unix Domain Socket support. (Pull #139)

### Fixed

- Always include the port on proxy CONNECT requests. (Pull #154)
- Fix `max_keepalive_connections` configuration. (Pull #153)
- Fixes behaviour in HTTP/1.1 where server disconnects can be used to signal the end of the response body. (Pull #164)

## 0.10.1 (August 7th, 2020)

- Include `max_keepalive_connections` on `AsyncHTTPProxy`/`SyncHTTPProxy` classes.

## 0.10.0 (August 7th, 2020)

The most notable change in the 0.10.0 release is that HTTP/2 support is now fully optional.

Use either `pip install httpcore` for HTTP/1.1 support only, or `pip install httpcore[http2]` for HTTP/1.1 and HTTP/2 support.

### Added

- HTTP/2 support becomes optional. (Pull #121, #130)
- Add `local_address=...` support. (Pull #100, #134)
- Add `PlainByteStream`, `IteratorByteStream`, `AsyncIteratorByteStream`. The `AsyncByteSteam` and `SyncByteStream` classes are now pure interface classes. (#133)
- Add `LocalProtocolError`, `RemoteProtocolError` exceptions. (Pull #129)
- Add `UnsupportedProtocol` exception. (Pull #128)
- Add `.get_connection_info()` method. (Pull #102, #137)
- Add better TRACE logs. (Pull #101)

### Changed

- `max_keepalive` is deprecated in favour of `max_keepalive_connections`. (Pull #140)

### Fixed

- Improve handling of server disconnects. (Pull #112)

## 0.9.1 (May 27th, 2020)

### Fixed

- Proper host resolution for sync case, including IPv6 support. (Pull #97)
- Close outstanding connections when connection pool is closed. (Pull #98)

## 0.9.0 (May 21th, 2020)

### Changed

- URL port becomes an `Optional[int]` instead of `int`. (Pull #92)

### Fixed

- Honor HTTP/2 max concurrent streams settings. (Pull #89, #90)
- Remove incorrect debug log. (Pull #83)

## 0.8.4 (May 11th, 2020)

### Added

- Logging via HTTPCORE_LOG_LEVEL and HTTPX_LOG_LEVEL environment variables
and TRACE level logging. (Pull #79)

### Fixed

- Reuse of connections on HTTP/2 in close concurrency situations. (Pull #81)

## 0.8.3 (May 6rd, 2020)

### Fixed

- Include `Host` and `Accept` headers on proxy "CONNECT" requests.
- De-duplicate any headers also contained in proxy_headers.
- HTTP/2 flag not being passed down to proxy connections.

## 0.8.2 (May 3rd, 2020)

### Fixed

- Fix connections using proxy forwarding requests not being added to the
connection pool properly. (Pull #70)

## 0.8.1 (April 30th, 2020)

### Changed

- Allow inherintance of both `httpcore.AsyncByteStream`, `httpcore.SyncByteStream` without type conflicts.

## 0.8.0 (April 30th, 2020)

### Fixed

- Fixed tunnel proxy support.

### Added

- New `TimeoutException` base class.

## 0.7.0 (March 5th, 2020)

- First integration with HTTPX.
