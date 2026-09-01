# zap-requests

An HTTP client library for [Zap](https://zaplang.xyz/), built directly on
`std/network` and `std/tls` (raw sockets) rather than wrapping `std/http`
(which only offers a GET-only, sentinel-error-coded helper).

Supports: all HTTP methods, custom headers, query parameters, raw/form
-urlencoded/JSON/multipart request bodies, redirect-following (including
relative `Location` headers), chunked transfer-encoding, and `!Error`
-based error handling throughout.

Not yet supported: connection reuse/keep-alive (every request opens and
closes its own socket), cookies/sessions, HTTP/2, compression, timeouts.

## Usage

```zap
import "std/io";
import "requests";

fun main() Int {
    let resp = requests.get("https://httpbin.org/get?hello=world") or err {
        io.println("request failed: " + err.message);
        return 1;
    };
    io.printInt(resp.status());
    io.println(resp.text());

    let posted = requests.Request.post("https://httpbin.org/post")
        .header("X-Custom", "zap-requests")
        .formField("name", "daniel")
        .send() or err {
            io.println("request failed: " + err.message);
            return 1;
        };
    io.println(posted.text());

    return 0;
}
```

See `src/main.zp` for a fuller runnable example covering plain HTTP, JSON
bodies, multipart uploads, redirects, and chunked responses (`thor run`).

## API

- Top-level shortcuts: `requests.get/post/put/delete/patch/head(url, ...)`
  all return `Response!Error`.
- `Request` (builder, chainable): `Request.get/post/put/delete/patch/head/
  options(url)`, then `.header(name, value)`, `.addHeader(name, value)`
  (allows duplicates), `.query(name, value)`, `.body(text, contentType)`,
  `.jsonBody(json.Value) Request!Error`, `.formField(name, value)`,
  `.multipartBody(MultipartBuilder)`, `.redirects(max)`, `.noRedirects()`,
  then `.send() Response!Error`.
- `MultipartBuilder`: `.addField(name, value)`, `.addFile(fieldName,
  filename, contentType, data)`, chainable, consumed by `.multipartBody(...)`.
- `Response`: `.status()`, `.statusText()`, `.headers() Headers`, `.text()`,
  `.json() json.Value!Error`, `.ok()`, `.isRedirect()`.
- `Headers`: `.set/.add/.get/.getOr/.has/.len/.nameAt/.valueAt`, case
  -insensitive, order-preserving, multi-value-capable.
- `urlEncode`/`urlDecode`, `parseUrl(raw) Url!Error`.

## Using this as a dependency

Same mechanism as `zap-yaml` (see its README) — no special layout needed:

```toml
[imports]
"@requests" = "path/to/zap-requests/src"
```

```zap
import "@requests/requests";
```
