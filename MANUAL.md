# Reqable Python Scripting API - Complete User Manual

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation & Setup](#2-installation--setup)
3. [Architecture & How It Works](#3-architecture--how-it-works)
4. [Quick Start](#4-quick-start)
5. [The Addon Script Structure](#5-the-addon-script-structure)
6. [API Reference](#6-api-reference)
   - [Context](#61-context)
   - [HttpRequest](#62-httprequest)
   - [HttpResponse](#63-httpresponse)
   - [HttpHeaders](#64-httpheaders)
   - [HttpQueries](#65-httpqueries)
   - [HttpBody](#66-httpbody)
   - [HttpMultipartBody](#67-httpmultipartbody)
   - [Connection & Address](#68-connection--address)
   - [App](#69-app)
   - [Highlight](#610-highlight)
7. [Cookbook: Complete Working Examples](#7-cookbook-complete-working-examples)
   - [Example 1: Pass-Through (No Modifications)](#example-1-pass-through-no-modifications)
   - [Example 2: Add Authentication Headers](#example-2-add-authentication-headers)
   - [Example 3: Rewrite API Endpoints](#example-3-rewrite-api-endpoints)
   - [Example 4: Modify JSON Request Bodies](#example-4-modify-json-request-bodies)
   - [Example 5: Modify JSON Response Bodies](#example-5-modify-json-response-bodies)
   - [Example 6: Generate Request Signatures (MD5)](#example-6-generate-request-signatures-md5)
   - [Example 7: HMAC-SHA256 Request Signing](#example-7-hmac-sha256-request-signing)
   - [Example 8: Share Data Between Request and Response](#example-8-share-data-between-request-and-response)
   - [Example 9: Conditional Routing by Host/Path](#example-9-conditional-routing-by-hostpath)
   - [Example 10: Mock API Responses](#example-10-mock-api-responses)
   - [Example 11: Add CORS Headers](#example-11-add-cors-headers)
   - [Example 12: Upload Files with Multipart Form Data](#example-12-upload-files-with-multipart-form-data)
   - [Example 13: Replace Response Body from a Local File](#example-13-replace-response-body-from-a-local-file)
   - [Example 14: Log and Highlight Traffic](#example-14-log-and-highlight-traffic)
   - [Example 15: Strip Tracking Parameters](#example-15-strip-tracking-parameters)
   - [Example 16: Inject Debug Headers and Environment Variables](#example-16-inject-debug-headers-and-environment-variables)
   - [Example 17: Rewrite Status Codes](#example-17-rewrite-status-codes)
   - [Example 18: Transform Response Content](#example-18-transform-response-content)
   - [Example 19: Rate-Limit Simulation](#example-19-rate-limit-simulation)
   - [Example 20: Full API Workflow - E-Commerce Checkout](#example-20-full-api-workflow---e-commerce-checkout)
8. [API Workflows: Chaining Multiple Scripts](#8-api-workflows-chaining-multiple-scripts)
9. [Testing Your Scripts](#9-testing-your-scripts)
10. [Troubleshooting & Common Pitfalls](#10-troubleshooting--common-pitfalls)
11. [Complete Class Reference Quick-Sheet](#11-complete-class-reference-quick-sheet)

---

## 1. Introduction

Reqable is an advanced HTTP debugging and testing tool. Its **Python Scripting API** lets you write Python scripts (called "addons") that intercept, inspect, and modify HTTP requests and responses as they flow through the Reqable proxy. This enables powerful use cases such as:

- **API testing** - Inject headers, modify payloads, change status codes
- **Request signing** - Compute HMAC/MD5 signatures and attach them automatically
- **Traffic manipulation** - Rewrite URLs, redirect endpoints, mock responses
- **Debugging** - Log requests, highlight traffic with colors, add comments
- **Security testing** - Strip or inject headers, manipulate authentication tokens
- **Development workflows** - Add CORS headers, swap environments, inject test data

This manual covers every class, method, and property in the API with concrete, copy-paste-ready examples.

---

## 2. Installation & Setup

### Install the Package

```bash
pip install reqable-scripting
```

Or with pip3:

```bash
pip3 install reqable-scripting
```

### Requirements

- Python >= 3.6
- Reqable application installed and running

### Setting Up in Reqable

1. Open Reqable and navigate to the **Addons** section
2. Reqable will create an `addons.py` file for you (or you can create one manually)
3. Write your `onRequest` and `onResponse` functions in this file
4. Enable the addon in Reqable's UI
5. Start capturing traffic - your scripts will execute automatically

The scripting framework is invoked by Reqable automatically via:
```bash
python main.py <request|response> <data_file.json>
```

You do not need to call this yourself - Reqable handles the lifecycle.

---

## 3. Architecture & How It Works

### Execution Flow

```
                    ┌──────────────┐
  Client Request ──>│   Reqable     │
                    │   Proxy       │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  addons.py   │
                    │ onRequest()  │──> Can modify: method, path, queries,
                    └──────┬───────┘    headers, body, trailers
                           │
                    ┌──────▼───────┐
                    │  Target      │
                    │  Server      │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  addons.py   │
                    │ onResponse() │──> Can modify: status code, headers,
                    └──────┬───────┘    body, trailers
                           │
                    ┌──────▼───────┐
                    │   Client     │
                    │   Receives   │
                    └──────────────┘
```

### Data Flow

1. **Reqable captures a request** and serializes it to a JSON file
2. **`onRequest()` runs** with a `Context` and `HttpRequest` object
3. Your script modifies the request (or leaves it unchanged)
4. The modified request is sent to the server
5. **The server responds** and Reqable serializes the response to JSON
6. **`onResponse()` runs** with a `Context` and `HttpResponse` object
7. Your script modifies the response (or leaves it unchanged)
8. The modified response is delivered to the client

### Callback File

After each hook runs, Reqable writes a `.cb` (callback) file containing:
- The modified request or response
- Updated environment variables
- Highlight color (if set)
- Comment text (if set)
- Shared data (for passing data between request and response hooks)

---

## 4. Quick Start

Create an `addons.py` file with the following minimal structure:

```python
from reqable import *

def onRequest(context, request):
    # Your request modification logic here
    return request

def onResponse(context, response):
    # Your response modification logic here
    return response
```

**Important rules:**
- Both functions **must** return their respective objects (`request` or `response`)
- The `from reqable import *` import gives you access to all classes
- You can use any Python standard library module (hashlib, json, re, etc.)

### Your First Modification

```python
from reqable import *

def onRequest(context, request):
    # Add a custom header to every request
    request.headers['X-Powered-By'] = 'Reqable'
    return request

def onResponse(context, response):
    return response
```

---

## 5. The Addon Script Structure

### Full Template (addons.py)

```python
# API Docs: https://reqable.com/docs/capture/addons
from reqable import *

def onRequest(context, request):
    # Print url to console
    # print('request url ' + context.url)

    # Update or add a query parameter
    # request.queries['foo'] = 'bar'

    # Update or add a http header
    # request.headers['foo'] = 'bar'

    # Replace http body with a text
    # request.body = 'Hello World'

    # Map with a local file
    # request.body.file('~/Desktop/body.json')

    # Convert to dict if the body is a JSON
    # request.body.jsonify()
    # Update the JSON content
    # request.body['foo'] = 'bar'

    # Done
    return request

def onResponse(context, response):
    # Update status code
    # response.code = 404

    # APIs are same as `onRequest`

    # Done
    return response
```

### Minimal Template (addons_mini.py)

```python
from reqable import *

def onRequest(context, request):
    return request

def onResponse(context, response):
    return response
```

---

## 6. API Reference

### 6.1 Context

The `Context` object provides metadata about the HTTP session, connection, and environment. It is passed as the first argument to both `onRequest` and `onResponse`.

#### Read-Only Properties

| Property | Type | Description |
|----------|------|-------------|
| `url` | `str` | Full request URL (e.g., `https://api.example.com/users?page=1`) |
| `scheme` | `str` | URL scheme: `"http"` or `"https"` |
| `host` | `str` | Host name (e.g., `"api.example.com"`) |
| `port` | `int` | Port number (e.g., `443`) |
| `id` | `int` | HTTP session ID |
| `timestamp` | `int` | Session timestamp in milliseconds |
| `connection` | `Connection` or `None` | TCP connection info (None for REST API) |
| `app` | `App` or `None` | Application info (None if not detected) |
| `env` | `dict` | Environment variables (key-value pairs) |
| `uid` | `str` | Unique HTTP ID in format `"ctime-cid-sid"` |

#### Read/Write Properties

| Property | Type | Description |
|----------|------|-------------|
| `highlight` | `Highlight` | Set a highlight color on this request in Reqable's UI |
| `comment` | `str` | Set a comment on this request in Reqable's UI |
| `shared` | `any` | Shared data between `onRequest` and `onResponse` hooks |

#### Deprecated Properties (still functional)

| Property | Use Instead |
|----------|-------------|
| `cid` | `connection.id` |
| `ctime` | `connection.timestamp` |
| `sid` | `id` |
| `stime` | `timestamp` |

#### Usage Examples

```python
def onRequest(context, request):
    # Access URL information
    print(f"URL: {context.url}")
    print(f"Scheme: {context.scheme}")
    print(f"Host: {context.host}")
    print(f"Port: {context.port}")

    # Access session info
    print(f"Session ID: {context.id}")
    print(f"Timestamp: {context.timestamp}")
    print(f"Unique ID: {context.uid}")

    # Access environment variables
    if context.env:
        for key, value in context.env.items():
            print(f"  {key} = {value}")

    # Access connection info (may be None for REST API)
    if context.connection:
        print(f"Connection ID: {context.connection.id}")
        print(f"Local: {context.connection.local.ip}:{context.connection.local.port}")
        print(f"Remote: {context.connection.remote.ip}:{context.connection.remote.port}")

    # Access application info
    if context.app:
        print(f"App: {context.app.name}")
        print(f"App ID: {context.app.id}")
        print(f"App Path: {context.app.path}")

    # Set highlight and comment
    context.highlight = Highlight.green
    context.comment = "Processed by addon"

    # Share data with the response hook
    context.shared = {"request_time": context.timestamp}

    return request
```

---

### 6.2 HttpRequest

Represents an HTTP request. Passed as the second argument to `onRequest`.

#### Read-Only Properties

| Property | Type | Description |
|----------|------|-------------|
| `protocol` | `str` | HTTP protocol (e.g., `"HTTP/1.1"`, `"h2"`) |
| `mime` | `str` or `None` | MIME type extracted from Content-Type header |

#### Read/Write Properties

| Property | Type | Description |
|----------|------|-------------|
| `method` | `str` | HTTP method (`"GET"`, `"POST"`, `"PUT"`, `"DELETE"`, etc.) |
| `path` | `str` | Request path without query string (e.g., `"/api/users"`) |
| `queries` | `HttpQueries` | URL query parameters |
| `headers` | `HttpHeaders` | HTTP request headers |
| `body` | `HttpBody` | Request body |
| `trailers` | `HttpHeaders` | HTTP trailers |
| `contentType` | `str` or `None` | Shortcut to get/set the `Content-Type` header |

#### Setting Properties - Multiple Input Formats

```python
def onRequest(context, request):
    # --- Method ---
    request.method = 'POST'

    # --- Path ---
    request.path = '/api/v2/users'

    # --- Queries: three ways to set ---
    # 1. From query string
    request.queries = 'page=1&limit=20'
    # 2. From dict
    request.queries = {'page': '1', 'limit': '20'}
    # 3. From list of tuples
    request.queries = [('page', '1'), ('limit', '20')]

    # --- Headers: three ways to set ---
    # 1. From list of strings
    request.headers = ['Content-Type: application/json', 'Authorization: Bearer token123']
    # 2. From dict
    request.headers = {'Content-Type': 'application/json', 'Authorization': 'Bearer token123'}
    # 3. From list of tuples
    request.headers = [('Content-Type', 'application/json'), ('Authorization', 'Bearer token123')]

    # --- Body: multiple ways to set ---
    # 1. From string
    request.body = '{"name": "John"}'
    # 2. From dict (auto-serialized to JSON string)
    request.body = {'name': 'John', 'age': 30}
    # 3. From bytes
    request.body = b'\x00\x01\x02\x03'

    # --- Content-Type shortcut ---
    request.contentType = 'application/json; charset=utf-8'

    return request
```

#### Serialization

```python
# Convert request to a JSON string
json_str = request.toJson()

# Convert request to a dict
data = request.serialize()
# Returns: {
#   'method': 'GET',
#   'path': '/api/users?page=1',
#   'protocol': 'HTTP/1.1',
#   'headers': ['Content-Type: application/json', ...],
#   'body': {'type': 1, 'payload': {...}},
#   'trailers': []
# }
```

---

### 6.3 HttpResponse

Represents an HTTP response. Passed as the second argument to `onResponse`.

#### Read-Only Properties

| Property | Type | Description |
|----------|------|-------------|
| `protocol` | `str` | HTTP protocol (e.g., `"HTTP/1.1"`, `"h2"`) |
| `message` | `str` or `None` | Status message (e.g., `"OK"`, `"Not Found"`) |
| `request` | `HttpRequest` | The original request that produced this response |
| `mime` | `str` or `None` | MIME type extracted from Content-Type header |

#### Read/Write Properties

| Property | Type | Description |
|----------|------|-------------|
| `code` | `int` | HTTP status code (100-600) |
| `headers` | `HttpHeaders` | Response headers |
| `body` | `HttpBody` | Response body |
| `trailers` | `HttpHeaders` | Response trailers |
| `contentType` | `str` or `None` | Shortcut to get/set the `Content-Type` header |

#### Usage

```python
def onResponse(context, response):
    # Read response info
    print(f"Status: {response.code} {response.message}")
    print(f"Protocol: {response.protocol}")
    print(f"Content-Type: {response.contentType}")
    print(f"MIME: {response.mime}")

    # Access the original request
    print(f"Request method: {response.request.method}")
    print(f"Request path: {response.request.path}")

    # Modify response
    response.code = 200
    response.headers['X-Custom'] = 'value'
    response.body = '{"status": "ok"}'

    return response
```

---

### 6.4 HttpHeaders

Manages HTTP headers with case-insensitive name lookups.

#### Creating Headers

```python
# Empty headers
headers = HttpHeaders()

# From list of strings
headers = HttpHeaders(['Content-Type: application/json', 'Accept: */*'])

# Using the factory method - from list of strings
headers = HttpHeaders.of(['Content-Type: application/json', 'Accept: */*'])

# From dict
headers = HttpHeaders.of({'Content-Type': 'application/json', 'Accept': '*/*'})

# From list of tuples
headers = HttpHeaders.of([('Content-Type', 'application/json'), ('Accept', '*/*')])
```

#### Reading Headers

```python
# Get a header value by name (case-insensitive)
content_type = headers['Content-Type']     # Returns: 'application/json'
content_type = headers['content-type']     # Also works! Returns: 'application/json'

# Returns None if header doesn't exist
auth = headers['Authorization']            # Returns: None

# Get by index (returns the full "name: value" string)
first_header = headers[0]                  # Returns: 'Content-Type: application/json'

# Get count
count = len(headers)                       # Returns: 2

# Iterate
for header in headers:
    print(header)  # Prints: "Content-Type: application/json", "Accept: */*"

# Find index of a header
idx = headers.index('content-type')        # Returns: 0 (case-insensitive)
idx = headers.index('nonexistent')         # Returns: -1

# Find all indexes (for duplicate headers)
indexes = headers.indexes('set-cookie')    # Returns: [2, 5] (all matching indexes)

# Convert to dict
d = headers.toDict()                       # {'Content-Type': 'application/json', 'Accept': '*/*'}

# Convert to JSON string
j = headers.toJson()                       # '{"Content-Type": "application/json", ...}'

# Get the raw entries list
entries = headers.entries                   # ['Content-Type: application/json', 'Accept: */*']

# Print headers
print(headers)                             # "['Content-Type: application/json', 'Accept: */*']"
```

#### Modifying Headers

```python
# Set/update a header (updates if exists, adds if not)
headers['Authorization'] = 'Bearer abc123'

# Add a header (always appends, even if name exists - allows duplicates)
headers.add('Set-Cookie', 'session=abc')
headers.add('Set-Cookie', 'theme=dark')

# Remove all headers with a name (case-insensitive)
headers.remove('Authorization')

# Clear all headers
headers.clear()
```

#### Important: Case-Insensitive Lookups

```python
headers = HttpHeaders(['Content-Type: application/json'])

# All of these return 'application/json':
headers['Content-Type']
headers['content-type']
headers['CONTENT-TYPE']
headers['Content-type']
```

---

### 6.5 HttpQueries

Manages URL query parameters with support for duplicate parameter names.

#### Creating Queries

```python
# Empty queries
queries = HttpQueries()

# Parse from query string
queries = HttpQueries.parse('foo=bar&page=1&limit=20')

# Using the factory method - from string
queries = HttpQueries.of('foo=bar&page=1&limit=20')

# From dict (values must be strings)
queries = HttpQueries.of({'foo': 'bar', 'page': '1'})

# From list of tuples
queries = HttpQueries.of([('foo', 'bar'), ('page', '1')])
```

#### Reading Queries

```python
queries = HttpQueries.parse('foo=bar&page=1&limit=20')

# Get a value by name
value = queries['foo']              # Returns: 'bar'
value = queries['nonexistent']      # Returns: None

# Get by index (returns tuple)
first = queries[0]                  # Returns: ('foo', 'bar')

# Get count
count = len(queries)                # Returns: 3

# Iterate
for name, value in queries:
    print(f"{name}={value}")

# Find index
idx = queries.index('page')        # Returns: 1
idx = queries.index('nonexistent') # Returns: -1

# Find all indexes (for duplicate parameters)
indexes = queries.indexes('tag')   # Returns: [2, 4]

# Convert to dict (last value wins for duplicates)
d = queries.toDict()               # {'foo': 'bar', 'page': '1', 'limit': '20'}

# Convert to JSON string
j = queries.toJson()               # '{"foo": "bar", "page": "1", "limit": "20"}'

# Reconstruct the query string
qs = queries.concat()              # 'foo=bar&page=1&limit=20' (URL-encoded)
qs = queries.concat(encode=False)  # 'foo=bar&page=1&limit=20' (not encoded)

# Get all entries
entries = queries.entries           # [('foo', 'bar'), ('page', '1'), ('limit', '20')]

# Print
print(queries)                     # "[('foo', 'bar'), ('page', '1'), ('limit', '20')]"
```

#### Modifying Queries

```python
# Set/update (updates first occurrence if exists, otherwise appends)
queries['page'] = '2'

# Add (always appends, even if name exists)
queries.add('tag', 'python')
queries.add('tag', 'scripting')     # Now has two 'tag' parameters

# Remove all parameters with a name
queries.remove('tag')

# Clear all
queries.clear()
```

#### URL Encoding

```python
queries = HttpQueries.parse('url=https%3A%2F%2Freqable.com')

# Values are automatically decoded
print(queries['url'])                  # 'https://reqable.com'

# concat() re-encodes by default
print(queries.concat())               # 'url=https%3A%2F%2Freqable.com'
print(queries.concat(encode=False))   # 'url=https://reqable.com'
```

#### Sorting Queries (useful for signatures)

```python
queries = HttpQueries.parse('c=3&a=1&b=2')

# sorted() works because queries are iterable tuples
sorted_queries = sorted(queries)       # [('a', '1'), ('b', '2'), ('c', '3')]
text = '&'.join(['='.join(q) for q in sorted_queries])
print(text)                            # 'a=1&b=2&c=3'
```

---

### 6.6 HttpBody

Manages HTTP request/response body content. Supports text, binary, and multipart types.

#### Body Types

| Type | Description | `payload` type |
|------|-------------|----------------|
| None | No body content | `None` |
| Text | Text or JSON string | `str` or `dict` (after `jsonify()`) |
| Binary | Raw bytes | `bytes` |
| Multipart | Multipart form data | `List[HttpMultipartBody]` |

#### Type-Checking Properties

```python
body.isNone       # True if no body
body.isText       # True if text/JSON
body.isBinary     # True if binary
body.isMultipart  # True if multipart
```

#### Creating Bodies

```python
# From string
body = HttpBody.of('Hello World')

# From dict (auto-serialized to JSON)
body = HttpBody.of({'name': 'John', 'age': 30})

# From bytes
body = HttpBody.of(b'\x89PNG\r\n\x1a\n')

# Empty body
body = HttpBody.of(None)
body = HttpBody.of()
```

#### Setting Body Content

```python
# Set to text
body.text('Hello World')

# Load text from file
body.textFromFile('/path/to/data.json')

# Set to binary from file
body.file('/path/to/image.png')

# Set to binary bytes
body.binary(b'\x00\x01\x02')

# Set to binary from file (alternative)
body.binary('/path/to/image.png')

# Set to multipart
body.multiparts([
    HttpMultipartBody.text('value1', name='field1'),
    HttpMultipartBody.file('/path/to/file.bin', name='upload', filename='data.bin')
])

# Clear the body
body.none()
```

#### Working with JSON Bodies

**Important: You must call `jsonify()` before accessing body content as a dictionary.**

```python
def onRequest(context, request):
    # Check if body is JSON
    if request.contentType and 'json' in request.contentType:
        # Parse the text payload into a dict
        request.body.jsonify()

        # Now you can read/write like a dict
        username = request.body['username']
        request.body['username'] = 'new_user'

        # Add new fields
        request.body['timestamp'] = 1234567890

        # Nested objects work too
        request.body['metadata'] = {'source': 'addon', 'version': 2}
        request.body['metadata']['source'] = 'updated_addon'

    return request
```

#### Text Operations

```python
# String replacement
body.text('Hello World')
body.replace('Hello', 'Hi')
print(body.payload)  # 'Hi World'

# Replace with count limit
body.text('aaa')
body.replace('a', 'b', 1)
print(body.payload)  # 'baa'
```

#### Reading Body Content

```python
# Get payload
payload = body.payload        # str, bytes, list, or None depending on type

# Get length
length = len(body)            # 0 if isNone

# Convert to string
text = str(body)              # Works for text and binary, raises for multipart

# Access binary bytes by index
byte = body[0]                # First byte (int) for binary bodies

# Access multipart parts by index
part = body[0]                # First HttpMultipartBody for multipart bodies
```

#### Writing Body to File

```python
# Save text body to file
body.writeFile('/path/to/output.txt')

# Save binary body to file
body.writeFile('/path/to/output.bin')
```

#### Directly Setting Body on Request/Response

```python
# These all use HttpBody.of() internally:
request.body = 'plain text'
request.body = {'key': 'value'}       # Auto-serialized to JSON string
request.body = b'\x00\x01\x02'

response.body = '{"status": "ok"}'
response.body = {'status': 'ok'}
response.body = b'\x89PNG\r\n'
```

---

### 6.7 HttpMultipartBody

Represents a single part within a multipart/form-data body. Inherits from `HttpBody`.

#### Creating Parts

```python
# Text part
part = HttpMultipartBody.text(
    'field_value',              # The text content
    name='field_name',          # Form field name
    filename='',                # Optional filename
    headers=None,               # Optional extra headers (list of strings)
    type='form-data',           # Disposition type
    charset='UTF-8'             # Character encoding
)

# File part
part = HttpMultipartBody.file(
    '/path/to/file.png',        # File path to read
    name='avatar',              # Form field name
    filename='profile.png',     # Filename sent to server
    headers=None,               # Optional extra headers
    type='form-data'            # Disposition type
)
```

#### Auto-Generated Headers

When using the factory methods, these headers are automatically added:
- `content-length`: Calculated from the content size
- `content-disposition`: Built from name, filename, and type parameters

```python
part = HttpMultipartBody.text('Hello', name='greeting', filename='hello.txt')
# Auto-generated headers:
#   content-length: 5
#   content-disposition: form-data; name="greeting"; filename="hello.txt"

part = HttpMultipartBody.text('Hello', name='greeting')
# Auto-generated headers:
#   content-length: 5
#   content-disposition: form-data; name="greeting"

part = HttpMultipartBody.text('Hello')
# Auto-generated headers:
#   content-length: 5
```

#### Properties

```python
# Access part headers
headers = part.headers              # HttpHeaders object

# Get/set the form field name
name = part.name                    # Reads from content-disposition
part.name = 'new_name'              # Updates content-disposition

# Get/set the filename
filename = part.filename            # Reads from content-disposition
part.filename = 'new_file.txt'      # Updates content-disposition

# Inherited from HttpBody:
part.isText                         # True/False
part.isBinary                       # True/False
part.payload                        # str or bytes
```

#### Complete Multipart Example

```python
def onRequest(context, request):
    # Create a multipart body with multiple parts
    parts = [
        HttpMultipartBody.text('John Doe', name='username'),
        HttpMultipartBody.text('john@example.com', name='email'),
        HttpMultipartBody.text('{"role": "admin"}', name='metadata'),
        HttpMultipartBody.file(
            '/path/to/avatar.png',
            name='avatar',
            filename='avatar.png',
            headers=['Content-Type: image/png']
        )
    ]

    request.body.multiparts(parts)
    return request
```

---

### 6.8 Connection & Address

Provides TCP connection details (available when traffic is captured via proxy, `None` for REST API mode).

#### Address

```python
# Access via context.connection.local or context.connection.remote
address = context.connection.local
print(address.ip)      # e.g., '192.168.1.100'
print(address.port)    # e.g., 54321
```

#### Connection

```python
conn = context.connection
if conn is not None:
    print(conn.id)                    # TCP connection ID
    print(conn.timestamp)             # Connection establishment timestamp (ms)
    print(conn.local.ip)              # Client IP
    print(conn.local.port)            # Client port
    print(conn.remote.ip)             # Server IP
    print(conn.remote.port)           # Server port
```

---

### 6.9 App

Provides information about the application making the request (when available).

```python
if context.app is not None:
    print(context.app.name)    # Application name (e.g., 'Chrome')
    print(context.app.id)      # Bundle ID / Package name (e.g., 'com.google.Chrome')
    print(context.app.path)    # Installation path (e.g., '/Applications/Chrome.app')
```

---

### 6.10 Highlight

Enum for color-coding traffic in Reqable's UI.

```python
context.highlight = Highlight.none          # No highlight (value: 0)
context.highlight = Highlight.red           # Red (value: 1)
context.highlight = Highlight.yellow        # Yellow (value: 2)
context.highlight = Highlight.green         # Green (value: 3)
context.highlight = Highlight.blue          # Blue (value: 4)
context.highlight = Highlight.teal          # Teal (value: 5)
context.highlight = Highlight.strikethrough # Strikethrough (value: 6)
```

---

## 7. Cookbook: Complete Working Examples

### Example 1: Pass-Through (No Modifications)

The simplest possible addon - does nothing, passes all traffic unmodified.

```python
from reqable import *

def onRequest(context, request):
    return request

def onResponse(context, response):
    return response
```

**Use case:** Starting template. Enable this addon to verify the scripting system works before adding logic.

---

### Example 2: Add Authentication Headers

Automatically inject authentication tokens into API requests.

```python
from reqable import *

def onRequest(context, request):
    # Only add auth to our API domain
    if context.host == 'api.mycompany.com':
        request.headers['Authorization'] = 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
        request.headers['X-API-Key'] = 'ak_live_1234567890abcdef'

    return request

def onResponse(context, response):
    return response
```

**Use case:** Testing authenticated APIs without manually adding tokens. When your token expires, just update it in one place.

---

### Example 3: Rewrite API Endpoints

Redirect traffic from one API version/environment to another.

```python
from reqable import *

def onRequest(context, request):
    # Redirect v1 API calls to v2
    if request.path.startswith('/api/v1/'):
        request.path = request.path.replace('/api/v1/', '/api/v2/')

    # Or redirect specific endpoints
    endpoint_map = {
        '/api/users': '/api/v2/accounts',
        '/api/posts': '/api/v2/articles',
        '/api/comments': '/api/v2/feedback',
    }

    if request.path in endpoint_map:
        request.path = endpoint_map[request.path]

    return request

def onResponse(context, response):
    return response
```

**Use case:** Testing new API versions with existing clients without code changes. Gradually migrating endpoints.

---

### Example 4: Modify JSON Request Bodies

Inject, modify, or remove fields from JSON request payloads.

```python
from reqable import *
import time

def onRequest(context, request):
    # Only process JSON requests to our API
    if context.host != 'api.example.com':
        return request

    if request.contentType and 'json' in request.contentType and request.body.isText:
        # Parse JSON string into a dict
        request.body.jsonify()

        # Add a timestamp to every request
        request.body['client_timestamp'] = int(time.time() * 1000)

        # Add device info
        request.body['device'] = {
            'platform': 'desktop',
            'app_version': '2.5.0',
            'os': 'macOS'
        }

        # Modify existing fields
        if 'debug' in str(request.body.payload):
            request.body['debug'] = True

    return request

def onResponse(context, response):
    return response
```

**Input body:** `{"username": "john", "action": "login"}`

**Output body:** `{"username": "john", "action": "login", "client_timestamp": 1686556322722, "device": {"platform": "desktop", "app_version": "2.5.0", "os": "macOS"}}`

---

### Example 5: Modify JSON Response Bodies

Alter API responses before they reach the client.

```python
from reqable import *

def onRequest(context, request):
    return request

def onResponse(context, response):
    # Only modify JSON responses
    if response.contentType and 'json' in response.contentType and response.body.isText:
        response.body.jsonify()

        # Inject additional data into user profile responses
        if response.request.path.startswith('/api/users/'):
            response.body['premium'] = True
            response.body['features'] = ['dark_mode', 'export', 'analytics']

        # Remove sensitive fields from all responses
        if isinstance(response.body.payload, dict):
            for field in ['internal_id', 'debug_info', 'trace_id']:
                if field in response.body.payload:
                    del response.body.payload[field]

    return response
```

**Use case:** Testing how your UI behaves with different data shapes. Enabling premium features during development. Stripping internal metadata before client delivery.

---

### Example 6: Generate Request Signatures (MD5)

Compute a signature from sorted query parameters and attach it as a header.

```python
from reqable import *
import hashlib

def onRequest(context, request):
    # Sort query parameters alphabetically
    queries = sorted(request.queries)

    # Join them into "key=value&key=value" format
    text = '&'.join(['='.join(query) for query in queries])

    # Compute MD5 hash
    algorithm = hashlib.md5()
    algorithm.update(text.encode(encoding='UTF-8'))
    signature = algorithm.hexdigest()

    # Attach as header
    request.headers['signature'] = signature

    return request

def onResponse(context, response):
    return response
```

**Given URL:** `https://api.example.com/data?hello=world&reqable=awesome&name=megatronking`

**Sorted parameters:** `hello=world&name=megatronking&reqable=awesome`

**Generated header:** `signature: 3e8c6c3b1cdca44384d0beaa487dbd21`

---

### Example 7: HMAC-SHA256 Request Signing

A production-grade request signing implementation with timestamp and nonce.

```python
from reqable import *
import hashlib
import hmac
import time
import uuid

API_SECRET = 'your-secret-key-here'

def onRequest(context, request):
    if context.host != 'api.secure-service.com':
        return request

    # Generate timestamp and nonce
    timestamp = str(int(time.time()))
    nonce = str(uuid.uuid4()).replace('-', '')

    # Build the string to sign:
    # METHOD\nPATH\nQUERY\nTIMESTAMP\nNONCE
    query_string = request.queries.concat(encode=False)
    string_to_sign = '\n'.join([
        request.method,
        request.path,
        query_string,
        timestamp,
        nonce
    ])

    # Compute HMAC-SHA256
    signature = hmac.new(
        API_SECRET.encode('utf-8'),
        string_to_sign.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()

    # Attach signing headers
    request.headers['X-Timestamp'] = timestamp
    request.headers['X-Nonce'] = nonce
    request.headers['X-Signature'] = signature

    return request

def onResponse(context, response):
    return response
```

---

### Example 8: Share Data Between Request and Response

Use `context.shared` to pass data from the request hook to the response hook.

```python
from reqable import *
import time

def onRequest(context, request):
    # Record when we saw the request
    context.shared = {
        'request_time': time.time(),
        'original_path': request.path,
        'original_method': request.method
    }

    return request

def onResponse(context, response):
    # Calculate round-trip time
    if context.shared and 'request_time' in context.shared:
        elapsed = time.time() - context.shared['request_time']
        elapsed_ms = int(elapsed * 1000)

        # Add timing header
        response.headers['X-Round-Trip-Ms'] = str(elapsed_ms)

        # Add comment with timing info
        context.comment = f"{context.shared['original_method']} {context.shared['original_path']} - {elapsed_ms}ms"

        # Highlight slow requests
        if elapsed_ms > 2000:
            context.highlight = Highlight.red
        elif elapsed_ms > 500:
            context.highlight = Highlight.yellow
        else:
            context.highlight = Highlight.green

    return response
```

---

### Example 9: Conditional Routing by Host/Path

Apply different transformations based on the target host and path.

```python
from reqable import *

def onRequest(context, request):
    # --- Production API ---
    if context.host == 'api.production.com':
        # Add production auth
        request.headers['Authorization'] = 'Bearer prod_token_abc'
        request.headers['X-Environment'] = 'production'
        context.highlight = Highlight.red
        context.comment = 'PRODUCTION'

    # --- Staging API ---
    elif context.host == 'api.staging.com':
        request.headers['Authorization'] = 'Bearer staging_token_xyz'
        request.headers['X-Environment'] = 'staging'
        context.highlight = Highlight.yellow
        context.comment = 'STAGING'

    # --- Dev API ---
    elif context.host == 'localhost' or context.host == '127.0.0.1':
        request.headers['X-Environment'] = 'development'
        request.headers['X-Debug'] = 'true'
        context.highlight = Highlight.blue
        context.comment = 'DEV'

    # --- Path-based routing ---
    if '/admin/' in request.path:
        request.headers['X-Admin-Access'] = 'true'
        context.highlight = Highlight.teal

    if request.path.startswith('/api/deprecated/'):
        context.highlight = Highlight.strikethrough
        context.comment = 'DEPRECATED ENDPOINT'

    return request

def onResponse(context, response):
    return response
```

---

### Example 10: Mock API Responses

Return fake responses without hitting the actual server. Useful for testing when backends are unavailable.

```python
from reqable import *
import json

def onRequest(context, request):
    return request

def onResponse(context, response):
    # Mock user profile endpoint
    if response.request.path == '/api/users/me':
        response.code = 200
        response.headers['Content-Type'] = 'application/json'
        response.body = json.dumps({
            'id': 42,
            'username': 'testuser',
            'email': 'test@example.com',
            'roles': ['admin', 'developer'],
            'avatar_url': 'https://example.com/avatars/42.jpg',
            'preferences': {
                'theme': 'dark',
                'language': 'en',
                'notifications': True
            }
        })
        context.highlight = Highlight.blue
        context.comment = 'MOCKED'

    # Mock product list endpoint
    elif response.request.path == '/api/products' and response.request.method == 'GET':
        response.code = 200
        response.headers['Content-Type'] = 'application/json'
        response.body = json.dumps({
            'products': [
                {'id': 1, 'name': 'Widget A', 'price': 9.99, 'stock': 150},
                {'id': 2, 'name': 'Widget B', 'price': 19.99, 'stock': 75},
                {'id': 3, 'name': 'Widget C', 'price': 29.99, 'stock': 0},
            ],
            'total': 3,
            'page': 1
        })
        context.highlight = Highlight.blue
        context.comment = 'MOCKED'

    # Simulate a 500 error for testing error handling
    elif response.request.path == '/api/flaky-endpoint':
        response.code = 500
        response.headers['Content-Type'] = 'application/json'
        response.body = json.dumps({
            'error': 'Internal Server Error',
            'message': 'Simulated failure for testing',
            'request_id': context.uid
        })
        context.highlight = Highlight.red
        context.comment = 'SIMULATED ERROR'

    return response
```

---

### Example 11: Add CORS Headers

Inject Cross-Origin Resource Sharing headers to all responses during development.

```python
from reqable import *

def onRequest(context, request):
    return request

def onResponse(context, response):
    # Add CORS headers to all responses
    response.headers['Access-Control-Allow-Origin'] = '*'
    response.headers['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS, PATCH'
    response.headers['Access-Control-Allow-Headers'] = 'Content-Type, Authorization, X-Requested-With'
    response.headers['Access-Control-Max-Age'] = '86400'
    response.headers['Access-Control-Allow-Credentials'] = 'true'

    # Handle preflight OPTIONS requests
    if response.request.method == 'OPTIONS':
        response.code = 204
        response.body = ''

    return response
```

**Use case:** Developing a frontend app that talks to an API on a different domain. Instead of configuring the server, inject CORS headers via the proxy.

---

### Example 12: Upload Files with Multipart Form Data

Construct multipart form-data requests programmatically.

```python
from reqable import *

def onRequest(context, request):
    if request.path == '/api/upload' and request.method == 'POST':
        # Build a multipart upload
        parts = [
            # Text form fields
            HttpMultipartBody.text('John Doe', name='author'),
            HttpMultipartBody.text('My vacation photos', name='description'),
            HttpMultipartBody.text('public', name='visibility'),

            # JSON metadata as a text field
            HttpMultipartBody.text(
                '{"tags": ["vacation", "beach"], "album": "Summer 2024"}',
                name='metadata'
            ),

            # File upload
            HttpMultipartBody.file(
                '/Users/john/Pictures/beach.jpg',
                name='photo',
                filename='beach.jpg',
                headers=['Content-Type: image/jpeg']
            ),

            # Another file
            HttpMultipartBody.file(
                '/Users/john/Pictures/sunset.png',
                name='photo',
                filename='sunset.png',
                headers=['Content-Type: image/png']
            ),
        ]

        request.body.multiparts(parts)

    return request

def onResponse(context, response):
    return response
```

---

### Example 13: Replace Response Body from a Local File

Map API responses to local JSON files for testing.

```python
from reqable import *

def onRequest(context, request):
    return request

def onResponse(context, response):
    # Map specific endpoints to local files
    file_map = {
        '/api/config': '/Users/dev/mocks/config.json',
        '/api/users/me': '/Users/dev/mocks/user-profile.json',
        '/api/feature-flags': '/Users/dev/mocks/feature-flags.json',
    }

    path = response.request.path

    if path in file_map:
        response.code = 200
        response.headers['Content-Type'] = 'application/json'
        response.body.textFromFile(file_map[path])
        context.highlight = Highlight.blue
        context.comment = f'MAPPED: {file_map[path]}'

    # Map binary responses (e.g., images)
    if path == '/api/avatar/default':
        response.code = 200
        response.headers['Content-Type'] = 'image/png'
        response.body.file('/Users/dev/mocks/default-avatar.png')
        context.comment = 'MAPPED: local avatar'

    return response
```

---

### Example 14: Log and Highlight Traffic

Color-code and annotate traffic for visual debugging.

```python
from reqable import *

def onRequest(context, request):
    # Color by method
    method_colors = {
        'GET': Highlight.green,
        'POST': Highlight.blue,
        'PUT': Highlight.yellow,
        'PATCH': Highlight.teal,
        'DELETE': Highlight.red,
    }

    if request.method in method_colors:
        context.highlight = method_colors[request.method]

    # Build a descriptive comment
    parts = [f"{request.method} {request.path}"]

    if request.contentType:
        parts.append(f"[{request.mime}]")

    if len(request.body) > 0:
        parts.append(f"({len(request.body)} bytes)")

    context.comment = ' '.join(parts)

    # Log to console
    print(f"[REQUEST] {context.url}")
    print(f"  Method: {request.method}")
    print(f"  Headers: {len(request.headers)}")
    print(f"  Body: {len(request.body)} bytes")

    return request

def onResponse(context, response):
    # Override color for errors
    if response.code >= 500:
        context.highlight = Highlight.red
        context.comment += f" -> {response.code} SERVER ERROR"
    elif response.code >= 400:
        context.highlight = Highlight.yellow
        context.comment += f" -> {response.code} CLIENT ERROR"
    else:
        context.comment += f" -> {response.code}"

    # Log to console
    print(f"[RESPONSE] {response.code} {response.message}")
    print(f"  Headers: {len(response.headers)}")
    print(f"  Body: {len(response.body)} bytes")

    return response
```

---

### Example 15: Strip Tracking Parameters

Remove analytics and tracking query parameters from URLs.

```python
from reqable import *

# Common tracking parameters to remove
TRACKING_PARAMS = [
    'utm_source', 'utm_medium', 'utm_campaign', 'utm_term', 'utm_content',
    'fbclid', 'gclid', 'dclid', 'msclkid',
    'mc_cid', 'mc_eid',
    '_ga', '_gl',
    'ref', 'referrer',
    'tracking_id', 'track',
]

def onRequest(context, request):
    removed = []

    for param in TRACKING_PARAMS:
        if request.queries[param] is not None:
            removed.append(param)
            request.queries.remove(param)

    if removed:
        context.highlight = Highlight.teal
        context.comment = f"Stripped: {', '.join(removed)}"

    return request

def onResponse(context, response):
    return response
```

**Before:** `https://example.com/page?id=42&utm_source=google&utm_medium=cpc&fbclid=abc123`

**After:** `https://example.com/page?id=42`

---

### Example 16: Inject Debug Headers and Environment Variables

Use environment variables from Reqable to control addon behavior dynamically.

```python
from reqable import *

def onRequest(context, request):
    # Read environment variables set in Reqable
    env = context.env or {}

    # Inject debug mode if env var is set
    if env.get('DEBUG_MODE') == 'true':
        request.headers['X-Debug'] = 'true'
        request.headers['X-Debug-Timestamp'] = str(context.timestamp)
        request.headers['X-Debug-Session'] = str(context.id)
        context.highlight = Highlight.teal

    # Use env vars for dynamic auth tokens
    auth_token = env.get('AUTH_TOKEN')
    if auth_token:
        request.headers['Authorization'] = f'Bearer {auth_token}'

    # Use Reqable's built-in random generators
    random_email = env.get('$randomEmail')
    if random_email and request.body.isText and 'email' in request.path:
        request.body.jsonify()
        request.body['email'] = random_email

    return request

def onResponse(context, response):
    return response
```

---

### Example 17: Rewrite Status Codes

Transform error responses into success responses for development testing.

```python
from reqable import *

def onRequest(context, request):
    return request

def onResponse(context, response):
    original_code = response.code

    # Turn 401/403 into 200 with a mock response (bypass auth during dev)
    if response.code in [401, 403]:
        response.code = 200
        response.headers['Content-Type'] = 'application/json'
        response.body = '{"authenticated": true, "role": "admin"}'
        context.comment = f"Auth bypassed (was {original_code})"
        context.highlight = Highlight.yellow

    # Turn 404 into a helpful message
    elif response.code == 404:
        response.code = 200
        response.headers['Content-Type'] = 'application/json'
        response.body = '{"data": null, "message": "Not found (mocked as 200)"}'
        context.comment = f"404 -> 200 (mocked)"
        context.highlight = Highlight.yellow

    # Turn 429 (rate limited) into 200 with retry-after header
    elif response.code == 429:
        response.code = 200
        response.headers['Content-Type'] = 'application/json'
        response.body = '{"data": [], "message": "Rate limit bypassed"}'
        context.comment = "Rate limit bypassed"
        context.highlight = Highlight.yellow

    return response
```

---

### Example 18: Transform Response Content

Modify specific fields in JSON responses using string replacement or JSON manipulation.

```python
from reqable import *
import re

def onRequest(context, request):
    return request

def onResponse(context, response):
    if not response.body.isText:
        return response

    # --- String replacement approach (fast, no parsing needed) ---
    # Replace all occurrences of staging URLs with production
    response.body.replace(
        'https://staging.example.com',
        'https://api.example.com'
    )

    # Replace placeholder images with real ones
    response.body.replace(
        'https://via.placeholder.com/150',
        'https://picsum.photos/150'
    )

    # --- JSON manipulation approach (structured, type-safe) ---
    if response.contentType and 'json' in response.contentType:
        if response.request.path.startswith('/api/products'):
            response.body.jsonify()
            payload = response.body.payload

            # Add a discount to all products
            if isinstance(payload, dict) and 'products' in payload:
                for product in payload['products']:
                    original_price = product.get('price', 0)
                    product['original_price'] = original_price
                    product['price'] = round(original_price * 0.9, 2)
                    product['discount'] = '10%'

    return response
```

---

### Example 19: Rate-Limit Simulation

Simulate rate limiting to test how your application handles 429 responses.

```python
from reqable import *

# Simple in-memory counter (resets when Reqable restarts)
request_count = 0
RATE_LIMIT = 5  # Allow 5 requests, then start returning 429

def onRequest(context, request):
    return request

def onResponse(context, response):
    global request_count

    # Only apply to specific API
    if context.host != 'api.example.com':
        return response

    request_count += 1

    if request_count > RATE_LIMIT:
        response.code = 429
        response.headers['Content-Type'] = 'application/json'
        response.headers['Retry-After'] = '60'
        response.headers['X-RateLimit-Limit'] = str(RATE_LIMIT)
        response.headers['X-RateLimit-Remaining'] = '0'
        response.body = '{"error": "rate_limit_exceeded", "message": "Too many requests"}'
        context.highlight = Highlight.red
        context.comment = f"RATE LIMITED (req #{request_count})"
    else:
        response.headers['X-RateLimit-Limit'] = str(RATE_LIMIT)
        response.headers['X-RateLimit-Remaining'] = str(RATE_LIMIT - request_count)
        context.comment = f"Request #{request_count}/{RATE_LIMIT}"

    return response
```

---

### Example 20: Full API Workflow - E-Commerce Checkout

A comprehensive example showing a complete API testing workflow that handles multiple endpoints, modifies both requests and responses, uses shared data, applies signatures, and visually annotates traffic.

```python
from reqable import *
import hashlib
import hmac
import json
import time
import uuid

# Configuration
API_HOST = 'api.shop.example.com'
API_SECRET = 'sk_test_abc123'
TEST_USER_TOKEN = 'eyJhbGciOiJIUzI1NiJ9.test_user_42'

def sign_request(method, path, body_str, timestamp, nonce):
    """Generate HMAC signature for the request."""
    message = f"{method}\n{path}\n{body_str}\n{timestamp}\n{nonce}"
    return hmac.new(
        API_SECRET.encode(),
        message.encode(),
        hashlib.sha256
    ).hexdigest()

def onRequest(context, request):
    # Only process our target API
    if context.host != API_HOST:
        return request

    # --- Global: Add auth and signing to all requests ---
    timestamp = str(int(time.time()))
    nonce = uuid.uuid4().hex

    request.headers['Authorization'] = f'Bearer {TEST_USER_TOKEN}'
    request.headers['X-Request-Id'] = str(uuid.uuid4())
    request.headers['X-Timestamp'] = timestamp
    request.headers['X-Nonce'] = nonce

    # Sign the request
    body_str = str(request.body) if not request.body.isNone else ''
    signature = sign_request(request.method, request.path, body_str, timestamp, nonce)
    request.headers['X-Signature'] = signature

    # Store request info for the response hook
    context.shared = {
        'start_time': time.time(),
        'endpoint': f"{request.method} {request.path}",
    }

    # --- Endpoint-specific modifications ---

    # 1. Add to Cart
    if request.path == '/api/cart/add' and request.method == 'POST':
        if request.body.isText:
            request.body.jsonify()
            request.body['quantity'] = request.body.get('quantity', 1)
            request.body['source'] = 'reqable_test'
        context.highlight = Highlight.blue
        context.comment = 'ADD TO CART'

    # 2. Checkout - inject test payment method
    elif request.path == '/api/checkout' and request.method == 'POST':
        if request.body.isText:
            request.body.jsonify()
            request.body['payment_method'] = 'test_card_visa'
            request.body['billing_address'] = {
                'line1': '123 Test Street',
                'city': 'Testville',
                'state': 'CA',
                'zip': '90210',
                'country': 'US'
            }
            request.body['test_mode'] = True
        context.highlight = Highlight.green
        context.comment = 'CHECKOUT'

    # 3. Search products - add test filters
    elif request.path == '/api/products/search':
        request.queries['limit'] = '50'
        request.queries['include_out_of_stock'] = 'true'
        if request.queries['q'] is None:
            request.queries['q'] = 'test'
        context.highlight = Highlight.teal
        context.comment = f"SEARCH: {request.queries['q']}"

    # 4. User profile - always request full data
    elif request.path == '/api/users/me':
        request.queries['include'] = 'orders,addresses,payment_methods'
        context.comment = 'PROFILE (full)'

    return request


def onResponse(context, response):
    if context.host != API_HOST:
        return response

    # Calculate response time
    if context.shared and 'start_time' in context.shared:
        elapsed_ms = int((time.time() - context.shared['start_time']) * 1000)
        response.headers['X-Elapsed-Ms'] = str(elapsed_ms)

        endpoint = context.shared.get('endpoint', '')

        # Update comment with timing
        if context.comment:
            context.comment += f" [{elapsed_ms}ms]"
        else:
            context.comment = f"{endpoint} [{elapsed_ms}ms]"

        # Highlight slow responses
        if elapsed_ms > 3000:
            context.highlight = Highlight.red

    # --- Handle error responses ---
    if response.code >= 400:
        # Log error details
        print(f"[ERROR] {response.code} on {response.request.method} {response.request.path}")
        if response.body.isText:
            print(f"  Body: {response.body.payload[:200]}")
        context.highlight = Highlight.red

    # --- Endpoint-specific response modifications ---

    # 1. Product list - add computed fields
    if response.request.path.startswith('/api/products') and response.code == 200:
        if response.body.isText and response.contentType and 'json' in response.contentType:
            response.body.jsonify()
            payload = response.body.payload
            if isinstance(payload, dict) and 'products' in payload:
                for product in payload['products']:
                    # Add availability label
                    stock = product.get('stock', 0)
                    if stock == 0:
                        product['availability'] = 'Out of Stock'
                    elif stock < 10:
                        product['availability'] = 'Low Stock'
                    else:
                        product['availability'] = 'In Stock'

    # 2. Checkout response - log order ID
    elif response.request.path == '/api/checkout' and response.code == 200:
        if response.body.isText:
            response.body.jsonify()
            order_id = response.body.payload.get('order_id', 'unknown')
            context.comment = f"ORDER CREATED: {order_id}"
            print(f"[CHECKOUT] Order {order_id} created successfully")

    return response
```

This example demonstrates:
- Host-based filtering
- Authentication header injection
- HMAC request signing
- Query parameter manipulation
- JSON body modification (both request and response)
- Shared data for timing
- Highlight and comment annotations
- Console logging
- Multi-endpoint handling in a single script

---

## 8. API Workflows: Chaining Multiple Scripts

While Reqable runs a single `addons.py` file, you can structure complex workflows within it using helper modules and conditional logic.

### Workflow Pattern: Modular Script Organization

Create helper modules alongside your `addons.py`:

**auth_helper.py:**
```python
import hashlib
import hmac
import time
import uuid

class AuthHelper:
    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret

    def sign(self, request):
        timestamp = str(int(time.time()))
        nonce = uuid.uuid4().hex

        string_to_sign = f"{request.method}\n{request.path}\n{timestamp}\n{nonce}"
        signature = hmac.new(
            self.api_secret.encode(),
            string_to_sign.encode(),
            hashlib.sha256
        ).hexdigest()

        request.headers['X-API-Key'] = self.api_key
        request.headers['X-Timestamp'] = timestamp
        request.headers['X-Nonce'] = nonce
        request.headers['X-Signature'] = signature

        return request
```

**mock_helper.py:**
```python
import json

MOCK_RESPONSES = {
    '/api/config': {
        'code': 200,
        'body': {
            'feature_flags': {'new_checkout': True, 'dark_mode': True},
            'api_version': '2.0',
            'maintenance': False
        }
    },
    '/api/health': {
        'code': 200,
        'body': {'status': 'healthy', 'uptime': 99.9}
    }
}

class MockHelper:
    @staticmethod
    def try_mock(response):
        path = response.request.path
        if path in MOCK_RESPONSES:
            mock = MOCK_RESPONSES[path]
            response.code = mock['code']
            response.headers['Content-Type'] = 'application/json'
            response.body = json.dumps(mock['body'])
            return True
        return False
```

**addons.py (main script):**
```python
from reqable import *
from auth_helper import AuthHelper
from mock_helper import MockHelper

# Initialize helpers
auth = AuthHelper('ak_test_123', 'sk_test_456')
mocker = MockHelper()

def onRequest(context, request):
    # Step 1: Apply authentication
    if context.host == 'api.example.com':
        request = auth.sign(request)

    # Step 2: Add common headers
    request.headers['X-Client'] = 'reqable-test'
    request.headers['Accept'] = 'application/json'

    # Step 3: Route-specific logic
    if '/admin/' in request.path:
        request.headers['X-Admin'] = 'true'
    elif '/public/' in request.path:
        request.headers.remove('Authorization')

    return request

def onResponse(context, response):
    # Step 1: Try mocks first
    if mocker.try_mock(response):
        context.highlight = Highlight.blue
        context.comment = 'MOCKED'
        return response

    # Step 2: Transform responses
    if response.code == 200 and response.body.isText:
        if response.contentType and 'json' in response.contentType:
            response.body.jsonify()
            if isinstance(response.body.payload, dict):
                response.body.payload['_reqable_processed'] = True

    # Step 3: Error handling
    if response.code >= 500:
        context.highlight = Highlight.red
        context.comment = f"ERROR {response.code}"

    return response
```

### Workflow Pattern: Environment Switching

```python
from reqable import *

# Define environment configurations
ENVIRONMENTS = {
    'dev': {
        'token': 'dev_token_123',
        'host_rewrite': None,
        'debug': True,
        'color': Highlight.blue,
    },
    'staging': {
        'token': 'staging_token_456',
        'host_rewrite': None,
        'debug': False,
        'color': Highlight.yellow,
    },
    'production': {
        'token': 'prod_token_789',
        'host_rewrite': None,
        'debug': False,
        'color': Highlight.red,
    }
}

# Change this to switch environments
ACTIVE_ENV = 'staging'

def onRequest(context, request):
    config = ENVIRONMENTS[ACTIVE_ENV]

    # Apply auth token
    request.headers['Authorization'] = f'Bearer {config["token"]}'

    # Enable debug mode
    if config['debug']:
        request.headers['X-Debug'] = 'true'
        request.queries['debug'] = '1'

    # Highlight with environment color
    context.highlight = config['color']
    context.comment = f'[{ACTIVE_ENV.upper()}] {request.method} {request.path}'

    return request

def onResponse(context, response):
    return response
```

### Workflow Pattern: Request/Response Pipeline

```python
from reqable import *

# --- Request Pipeline Steps ---

def step_add_correlation_id(context, request):
    """Add a correlation ID for request tracing."""
    import uuid
    correlation_id = str(uuid.uuid4())
    request.headers['X-Correlation-Id'] = correlation_id
    context.shared = context.shared or {}
    context.shared['correlation_id'] = correlation_id
    return request

def step_normalize_headers(context, request):
    """Ensure standard headers are present."""
    if request.headers['Accept'] is None:
        request.headers['Accept'] = 'application/json'
    if request.headers['Accept-Encoding'] is None:
        request.headers['Accept-Encoding'] = 'gzip, deflate'
    return request

def step_inject_test_data(context, request):
    """Inject test-specific data into requests."""
    if request.body.isText and request.contentType and 'json' in request.contentType:
        request.body.jsonify()
        if isinstance(request.body.payload, dict):
            request.body.payload['_test'] = True
            request.body.payload['_test_session'] = context.uid
    return request

# --- Response Pipeline Steps ---

def step_log_response(context, response):
    """Log response summary."""
    print(f"[{response.code}] {response.request.method} {response.request.path}")
    return response

def step_validate_response(context, response):
    """Check for common issues in responses."""
    if response.code == 200 and response.body.isNone:
        context.comment = 'WARNING: 200 with empty body'
        context.highlight = Highlight.yellow
    return response

# --- Main Addon ---

def onRequest(context, request):
    # Run all request pipeline steps in order
    request = step_add_correlation_id(context, request)
    request = step_normalize_headers(context, request)
    request = step_inject_test_data(context, request)
    return request

def onResponse(context, response):
    # Run all response pipeline steps in order
    response = step_log_response(context, response)
    response = step_validate_response(context, response)
    return response
```

---

## 9. Testing Your Scripts

### Running the Test Suite

The project includes a comprehensive test suite. Run all tests:

```bash
cd test
bash test.sh
```

Or run individual test files:

```bash
cd test
export PYTHONPATH="../reqable"
python3 reqable_header_test.py
python3 reqable_query_test.py
python3 reqable_body_test.py
python3 reqable_request_test.py
python3 reqable_response_test.py
python3 reqable_context_test.py
python3 reqable_multipart_body_test.py
```

### Integration Test Cases

Navigate into each case directory to run integration tests:

```bash
cd test/case1
export PYTHONPATH="../../reqable"
python3 reqable_main_test.py
```

### Writing Your Own Tests

Create test files that simulate the Reqable lifecycle:

```python
import unittest
import json
import sys
import os

# Add the reqable module to path
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'reqable'))

from reqable import *

class MyAddonTest(unittest.TestCase):

    def test_auth_header_injection(self):
        """Test that auth headers are added to matching requests."""
        # Import your addon
        import addons

        # Create a mock context
        context = Context({
            'url': 'https://api.mycompany.com/users',
            'scheme': 'https',
            'host': 'api.mycompany.com',
            'port': 443,
            'id': 1,
            'timestamp': 1686556322722,
            'connection': None,
        })

        # Create a mock request
        request = HttpRequest({
            'method': 'GET',
            'path': '/users',
            'protocol': 'HTTP/1.1',
            'headers': ['Accept: application/json'],
            'body': {'type': 0, 'payload': None},
            'trailers': []
        })

        # Run the addon
        result = addons.onRequest(context, request)

        # Verify the result
        self.assertIsNotNone(result)
        self.assertIsNotNone(result.headers['Authorization'])
        self.assertTrue(result.headers['Authorization'].startswith('Bearer '))

    def test_json_body_modification(self):
        """Test that JSON bodies are correctly modified."""
        import addons

        context = Context({
            'url': 'https://api.example.com/data',
            'scheme': 'https',
            'host': 'api.example.com',
            'port': 443,
            'id': 2,
            'timestamp': 1686556322722,
            'connection': None,
        })

        request = HttpRequest({
            'method': 'POST',
            'path': '/data',
            'protocol': 'HTTP/1.1',
            'headers': ['Content-Type: application/json'],
            'body': {
                'type': 1,
                'payload': {
                    'text': '{"username": "test"}',
                    'charset': 'UTF-8'
                }
            },
            'trailers': []
        })

        result = addons.onRequest(context, request)

        # Verify the body was modified
        self.assertTrue(result.body.isText)

    def test_response_status_override(self):
        """Test that error responses are converted."""
        import addons

        context = Context({
            'url': 'https://api.example.com/resource',
            'scheme': 'https',
            'host': 'api.example.com',
            'port': 443,
            'id': 3,
            'timestamp': 1686556322722,
            'connection': None,
        })

        response = HttpResponse({
            'request': {
                'method': 'GET',
                'path': '/resource',
                'protocol': 'HTTP/1.1',
            },
            'code': 401,
            'message': 'Unauthorized',
            'protocol': 'HTTP/1.1',
            'headers': ['Content-Type: application/json'],
            'body': {
                'type': 1,
                'payload': {
                    'text': '{"error": "unauthorized"}',
                    'charset': 'UTF-8'
                }
            },
            'trailers': []
        })

        result = addons.onResponse(context, response)
        # Verify assertions based on your addon logic
        self.assertIsNotNone(result)

    def test_shared_data_flow(self):
        """Test that shared data passes from request to response."""
        import addons

        context = Context({
            'url': 'https://api.example.com/test',
            'scheme': 'https',
            'host': 'api.example.com',
            'port': 443,
            'id': 4,
            'timestamp': 1686556322722,
            'connection': None,
            'shared': None,
        })

        request = HttpRequest({
            'method': 'GET',
            'path': '/test',
            'protocol': 'HTTP/1.1',
        })

        # Run onRequest - should set shared data
        addons.onRequest(context, request)
        self.assertIsNotNone(context.shared)

        response = HttpResponse({
            'request': {
                'method': 'GET',
                'path': '/test',
                'protocol': 'HTTP/1.1',
            },
            'code': 200,
            'message': 'OK',
            'protocol': 'HTTP/1.1',
        })

        # Run onResponse - should use shared data
        result = addons.onResponse(context, response)
        self.assertIsNotNone(result)

    def test_query_manipulation(self):
        """Test that query parameters are correctly modified."""
        request = HttpRequest({
            'method': 'GET',
            'path': '/search?q=test&page=1',
            'protocol': 'HTTP/1.1',
        })

        # Verify query parsing
        self.assertEqual(request.queries['q'], 'test')
        self.assertEqual(request.queries['page'], '1')

        # Modify queries
        request.queries['limit'] = '50'
        request.queries.remove('page')

        # Verify changes
        self.assertEqual(len(request.queries), 2)
        self.assertEqual(request.queries['limit'], '50')
        self.assertIsNone(request.queries['page'])

        # Verify serialization includes queries
        serialized = request.serialize()
        self.assertIn('limit=50', serialized['path'])

    def test_headers_case_insensitive(self):
        """Test that header lookups are case-insensitive."""
        headers = HttpHeaders([
            'Content-Type: application/json',
            'X-Custom-Header: value123'
        ])

        self.assertEqual(headers['content-type'], 'application/json')
        self.assertEqual(headers['Content-Type'], 'application/json')
        self.assertEqual(headers['CONTENT-TYPE'], 'application/json')
        self.assertEqual(headers['x-custom-header'], 'value123')

if __name__ == '__main__':
    unittest.main()
```

---

## 10. Troubleshooting & Common Pitfalls

### Pitfall 1: Forgetting to Return the Object

```python
# WRONG - returns None, Reqable ignores the modifications
def onRequest(context, request):
    request.headers['Foo'] = 'bar'
    # Missing return!

# CORRECT
def onRequest(context, request):
    request.headers['Foo'] = 'bar'
    return request
```

### Pitfall 2: Forgetting to Call jsonify()

```python
# WRONG - raises Exception
def onRequest(context, request):
    request.body['key'] = 'value'  # Error: Did you forget to call jsonify()?
    return request

# CORRECT
def onRequest(context, request):
    request.body.jsonify()          # Parse text into dict first
    request.body['key'] = 'value'   # Now this works
    return request
```

### Pitfall 3: Query Values Must Be Strings

```python
# WRONG - raises Exception
request.queries = {'page': 1, 'limit': 20}

# CORRECT - all values must be strings
request.queries = {'page': '1', 'limit': '20'}
```

### Pitfall 4: Status Code Range

```python
# WRONG - raises Exception
response.code = 99     # Must be >= 100
response.code = 601    # Must be <= 600
response.code = '200'  # Must be int

# CORRECT
response.code = 200
response.code = 404
response.code = 503
```

### Pitfall 5: Setting Method or Path to Empty String

```python
# WRONG - raises Exception
request.method = ''
request.path = ''

# CORRECT
request.method = 'POST'
request.path = '/api/endpoint'
```

### Pitfall 6: Modifying Body Content-Type Mismatch

```python
# Be careful: setting body to a dict serializes it as JSON text
request.body = {'key': 'value'}  # Sets as text type, payload = '{"key": "value"}'
# Make sure Content-Type matches:
request.contentType = 'application/json'
```

### Pitfall 7: Accessing Response Request

```python
# In onResponse, you can access the original request:
def onResponse(context, response):
    # This works - read the request that generated this response
    original_method = response.request.method
    original_path = response.request.path
    original_headers = response.request.headers

    # But modifications to response.request are serialized back too
    # so be careful about unintended changes
    return response
```

### Pitfall 8: Binary Body String Representation

```python
# Binary bodies can't be meaningfully printed
body = HttpBody.of(b'\x89PNG\r\n')
print(str(body))  # Prints bytes representation: b'\x89PNG\r\n'

# Multipart bodies raise an exception when converted to string
body.multiparts([...])
print(str(body))  # Raises: 'Unsupported str for multipart body'
# Use repr() instead:
print(repr(body))  # Prints: 'Multipart 2 body'
```

### Pitfall 9: Header Duplicate Behavior

```python
headers = HttpHeaders(['Set-Cookie: a=1', 'Set-Cookie: b=2'])

# __getitem__ returns the FIRST match only
print(headers['Set-Cookie'])  # 'a=1' (not 'b=2')

# Use indexes() to find all
idxs = headers.indexes('Set-Cookie')  # [0, 1]

# add() always appends (creates duplicates)
headers.add('Set-Cookie', 'c=3')  # Now has 3 Set-Cookie headers

# __setitem__ updates the FIRST match
headers['Set-Cookie'] = 'new=value'  # Updates index 0, leaves others
```

---

## 11. Complete Class Reference Quick-Sheet

```
┌─────────────────────────────────────────────────────────────────────┐
│                           Context                                    │
├─────────────────────────────────────────────────────────────────────┤
│ Read-only:  url, scheme, host, port, id, timestamp, connection,     │
│             app, env, uid                                            │
│ Read/Write: highlight, comment, shared                               │
│ Deprecated: cid, ctime, sid, stime                                   │
│ Methods:    toJson()                                                 │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         HttpRequest                                  │
├─────────────────────────────────────────────────────────────────────┤
│ Read-only:  protocol, mime                                           │
│ Read/Write: method, path, queries, headers, body, trailers,         │
│             contentType                                              │
│ Methods:    serialize(), toJson()                                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         HttpResponse                                 │
├─────────────────────────────────────────────────────────────────────┤
│ Read-only:  protocol, message, request, mime                         │
│ Read/Write: code, headers, body, trailers, contentType               │
│ Methods:    serialize(), toJson()                                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         HttpHeaders                                  │
├─────────────────────────────────────────────────────────────────────┤
│ Read-only:  entries                                                  │
│ Factory:    HttpHeaders.of(data)                                     │
│ Methods:    add(name, value), remove(name), clear(),                 │
│             index(name), indexes(name), toDict(), toJson()           │
│ Operators:  [name] -> value, [index] -> "name: value",              │
│             [name] = value, len(), iter(), str()                     │
│ Note:       Case-insensitive name lookups                            │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         HttpQueries                                  │
├─────────────────────────────────────────────────────────────────────┤
│ Read-only:  entries, origin                                          │
│ Factory:    HttpQueries.parse(query_str), HttpQueries.of(data)       │
│ Methods:    add(name, value), remove(name), clear(),                 │
│             index(name), indexes(name), concat(encode=True),         │
│             toDict(), toJson()                                       │
│ Operators:  [name] -> value, [index] -> (name, value),              │
│             [name] = value, len(), iter(), str()                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                          HttpBody                                    │
├─────────────────────────────────────────────────────────────────────┤
│ Read-only:  type, payload, isNone, isText, isBinary, isMultipart     │
│ Factory:    HttpBody.of(data), HttpBody.parse(dict)                  │
│ Setters:    none(), text(str), textFromFile(path), file(path),       │
│             binary(bytes|path), multiparts(list)                     │
│ Methods:    jsonify(), replace(old, new, count=-1),                  │
│             writeFile(path), serialize()                             │
│ Operators:  [key] (after jsonify), [key]=val, len(), iter(), str()   │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                      HttpMultipartBody                                │
│                    (extends HttpBody)                                 │
├─────────────────────────────────────────────────────────────────────┤
│ Read/Write: headers, name, filename                                  │
│ Factory:    HttpMultipartBody.text(...), HttpMultipartBody.file(...)  │
│ Methods:    serialize() (+ all inherited from HttpBody)               │
└─────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────┐  ┌───────────────────────────────┐
│         Connection            │  │          Address              │
├───────────────────────────────┤  ├───────────────────────────────┤
│ id, timestamp, local, remote  │  │ ip, port                      │
│ serialize()                   │  │ serialize()                   │
└───────────────────────────────┘  └───────────────────────────────┘

┌───────────────────────────────┐  ┌───────────────────────────────┐
│            App                │  │         Highlight             │
├───────────────────────────────┤  ├───────────────────────────────┤
│ name, id, path                │  │ none=0, red=1, yellow=2,      │
│ serialize()                   │  │ green=3, blue=4, teal=5,      │
└───────────────────────────────┘  │ strikethrough=6               │
                                   └───────────────────────────────┘

Legacy Aliases (deprecated):
  CaptureApp = App
  CaptureContext = Context
  CaptureHttpQueries = HttpQueries
  CaptureHttpHeaders = HttpHeaders
  CaptureHttpBody = HttpBody
  CaptureHttpMultipartBody = HttpMultipartBody
  CaptureHttpRequest = HttpRequest
  CaptureHttpResponse = HttpResponse
```

---

*Generated for reqable-scripting v1.2.0 | Python >= 3.6 | MIT License*
*Official documentation: https://reqable.com/docs/capture/addons*
*GitHub: https://github.com/reqable/python-scripting-api*
