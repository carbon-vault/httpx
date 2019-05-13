# HTTPCore

<a href="https://travis-ci.org/encode/httpcore">
    <img src="https://travis-ci.org/encode/httpcore.svg?branch=master" alt="Build Status">
</a>
<a href="https://codecov.io/gh/encode/httpcore">
    <img src="https://codecov.io/gh/encode/httpcore/branch/master/graph/badge.svg" alt="Coverage">
</a>
<a href="https://pypi.org/project/httpcore/">
    <img src="https://badge.fury.io/py/httpcore.svg" alt="Package version">
</a>

## Feature support

* `HTTP/1.1` and `HTTP/2` Support.
* `async`/`await` support for non-blocking HTTP requests.
* Strict timeouts everywhere by default.
* Fully type annotated.
* 100% test coverage. *TODO - Almost there.*

Plus all the standard features of requests...

* International Domains and URLs
* Keep-Alive & Connection Pooling
* Sessions with Cookie Persistence *TODO*
* Browser-style SSL Verification
* Basic/Digest Authentication *TODO*
* Elegant Key/Value Cookies *TODO*
* Automatic Decompression
* Automatic Content Decoding
* Unicode Response Bodies
* Multipart File Uploads *TODO - Request content currently supports URL encoded data, bytes, or async byte iterators.*
* HTTP(S) Proxy Support *TODO*
* Connection Timeouts
* Streaming Downloads
* .netrc Support *TODO*
* Chunked Requests

## Usage

Making a request:

```python
>>> import httpcore
>>> client = httpcore.Client()
>>> response = client.get('https://example.com')
>>> response.status_code
<StatusCode.ok: 200>
>>> response.protocol
'HTTP/2'
>>> response.text
'<!doctype html>\n<html>\n<head>\n<title>Example Domain</title>\n...'
```

Alternatively, async requests:

**Note**: Use `ipython` to try this from the console, since it supports `await`.

```python
>>> import httpcore
>>> client = httpcore.AsyncClient()
>>> response = await client.get('https://example.com')
>>> response.status_code
<StatusCode.ok: 200>
>>> response.protocol
'HTTP/2'
>>> response.text
'<!doctype html>\n<html>\n<head>\n<title>Example Domain</title>\n...'
```

---

## Dependencies

* `h11` - HTTP/1.1 support.
* `h2` - HTTP/2 support.
* `certifi` - SSL certificates.
* `chardet` - Fallback auto-detection for response encoding.
* `idna` - Internationalized domain name support.
* `rfc3986` - URL parsing & normalization.
* `brotlipy` - Decoding for "brotli" compressed responses. *(Optional)*

A huge amount of credit is due to `requests` for the API layout that
much of this work follows, as well as to `urllib3` for plenty of design
inspiration around the lower level networking details.

---

## API Reference

### `Client`

*An HTTP client, with connection pooling, redirects, cookie persistence, etc.*

```python
>>> client = Client()
>>> response = client.get('https://example.org')
```

* `def __init__([ssl], [timeout], [pool_limits], [max_redirects], [dispatch])`
* `def .request(method, url, [content], [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .get(url, [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .options(url, [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .head(url, [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .post(url, [content], [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .put(url, [content], [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .patch(url, [content], [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .delete(url, [content], [query_params], [headers], [stream], [allow_redirects], [ssl], [timeout])`
* `def .prepare_request(request)`
* `def .send(request, [stream], [allow_redirects], [ssl], [timeout])`
* `def .close()`

### `Response`

*An HTTP response.*

* `def __init__(...)`
* `.status_code` - **int**
* `.reason_phrase` - **str**
* `.protocol` - `"HTTP/2"` or `"HTTP/1.1"`
* `.url` - **URL**
* `.headers` - **Headers**
* `.content` - **bytes**
* `.text` - **str**
* `.encoding` - **str**
* `.is_redirect` - **bool**
* `.request` - **Request**
* `.cookies` - **Cookies** *TODO*
* `.history` - **List[Response]**
* `def .raise_for_status()` - **None**
* `def .json()` - **Any** *TODO*
* `def .read()` - **bytes**
* `def .stream()` - **bytes iterator**
* `def .raw()` - **bytes iterator**
* `def .close()` - **None**
* `def .next()` - **Response**

### `Request`

*An HTTP request. Can be constructed explicitly for more control over exactly
what gets sent over the wire.*

```python
>>> request = Request("GET", "https://example.org", headers={'host': 'example.org'})
>>> response = client.send(request)
```

* `def __init__(method, url, query_params, content, headers)`
* `.method` - **str** (Uppercased)
* `.url` - **URL**
* `.content` - **byte** or **byte async iterator**
* `.headers` - **Headers**

### `URL`

*A normalized, IDNA supporting URL.*

```python
>>> url = URL("https://example.org/")
>>> url.host
'example.org'
```

* `def __init__(url, allow_relative=False, query_params=None)`
* `.scheme` - **str**
* `.authority` - **str**
* `.host` - **str**
* `.port` - **int**
* `.path` - **str**
* `.query` - **str**
* `.full_path` - **str**
* `.fragment` - **str**
* `.is_ssl` - **bool**
* `.origin` - **Origin**
* `.is_absolute_url` - **bool**
* `.is_relative_url` - **bool**
* `def .copy_with([scheme], [authority], [path], [query], [fragment])` - **URL**
* `def .resolve_with(url)` - **URL**

### `Origin`

*A normalized, IDNA supporting set of scheme/host/port info.*

```python
>>> Origin('https://example.org') == Origin('HTTPS://EXAMPLE.ORG:443')
True
```

* `def __init__(url)`
* `.is_ssl` - **bool**
* `.host` - **str**
* `.port` - **int**

### `Headers`

*A case-insensitive multi-dict.*

```python
>>> headers = Headers({'Content-Type': 'application/json'})
>>> headers['content-type']
'application/json'
```

* `def __init__(headers)`

___

## Alternate backends

### `AsyncClient`

An asyncio client.

### `TrioClient`

*TODO*

---

## The Stack

There are two main layers in the stack. The client handles redirection,
cookie persistence (TODO), and authentication (TODO). The dispatcher
handles sending the actual request and getting the response.

* `Client` - Redirect, authentication, cookies etc.
* `ConnectionPool(Dispatcher)` - Connection pooling & keep alive.
  * `HTTPConnection` - A single connection.
    * `HTTP11Connection` - A single HTTP/1.1 connection.
    * `HTTP2Connection` - A single HTTP/2 connection, with multiple streams.
