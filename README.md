# webcrystal

webcrystal is:

1. An HTTP proxy and web service that saves every web page accessed through it to disk.
2. An on-disk archival format for storing websites.

webcrystal is intended as a tool for archiving websites.

## Features

* Compact package: One .py file. Only one dependency (`urllib3`).
* A simple documented archival format.
* Nearly 100% code coverage.
* Friendly MIT license.

## Usage

To start the proxy run a command like:

```
./webcrystal.py 9227 xkcd.wbcr http://xkcd.com/
```

Then you can visit <http://localhost:9227/> to have the same effect as visiting <http://xkcd.com/> directly, except that all requests are archived in `xkcd.wbcr/`.

When you access an HTTP resource through the webcrystal proxy for the first time, it will be fetched from the origin HTTP server and archived locally. All subsequent requests for the same resource will be returned from the archive.


## CLI

### Starting the Proxy

```
./webcrystal.py [--help] [--quiet] <port> <archive_dirpath> [<default_origin_domain>]
```


## API

While the proxy is running, it responds to the following API endpoints.

Notice that GET is an accepted method for all endpoints, so that they can be easily requested using a regular web browser.

### `GET,HEAD /`

Redirects to the home page of the default origin domain if it is known. Returns:

* HTTP 404 (Not Found) if no default origin domain is specified (the default) or
* HTTP 307 (Temporary Redirect) to the default origin domain if it is specified.

### `GET,HEAD /_/http[s]/__PATH__`

If in online mode (the default):

* The requested resource will be fetched from the origin server and added to the archive if:
    * (1) it is not already archived,
    * (2) a `Cache-Control=no-cache` header is specified, or
    * (3) a `Pragma=no-cache` header is specified.
* The newly archived resource will be returned to the client, with all URLs in HTTP headers and content rewritten to point to the proxy.

If in offline mode:

* If the resource is in the archive, it will be returned to the client, with all URLs in HTTP headers and content rewritten to point to the proxy.
* If the resource is not in the archive, an HTTP 503 (Service Unavailable) response will be returned, with an HTML page that provides a link to the online version of the content.

### `POST,GET /_online`

Switches the proxy to online mode.

### `POST,GET /_offline`

Switches the proxy to offline mode.

### `POST,GET /_refresh/http[s]/__PATH__`

Refetches the specified URL from the origin server using the same request headers as the last time it was fetched. Returns:

* HTTP 200 (OK) if successful or
* HTTP 404 (Not Found) if the specified URL was not in the archive.

### `POST,GET /_delete/http[s]/__PATH__`

Deletes the specified URL in the archive. Returns:

* HTTP 200 (OK) if successful or
* HTTP 404 (Not Found) if the specified URL was not in the archive.


## Archival Format

When this proxy is started with a command like:

```
./webcrystal.py 9227 xkcd.wbcr
```

It creates an archive in the directory `xkcd.wbcr/` in the following format:


### `xkcd.wbcr/index.txt`

* Lists the URL of each archived HTTP resource, one per line.
* UTF-8 encoded text file with Unix line endings (`\n`).

Example:

```
http://xkcd.com/
http://xkcd.com/s/b0dcca.css
http://xkcd.com/1645/
```

The preceding example archive contains 3 HTTP resources, numbered #1, #2, and #3.


### `xkcd.wbcr/1.request_headers.json`

* Contains the HTTP request headers sent to the origin HTTP server to obtain HTTP resource #1.
* UTF-8 encoded JSON file.

Example:

```
{"Accept-Language": "en-us", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8", "Host": "xkcd.com", "Accept-Encoding": "gzip, deflate", "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/601.4.4 (KHTML, like Gecko) Version/9.0.3 Safari/601.4.4"}
```


### `xkcd.wbcr/1.response_headers.json`

* Contains the HTTP response headers received from the origin HTTP server when obtaining HTTP resource #1.
* UTF-8 encoded JSON file.
* Contains an internal "X-Status-Code" header that indicates the HTTP status code received from the origin HTTP server.

Example:

```
{"Cache-Control": "public", "Connection": "keep-alive", "Accept-Ranges": "bytes", "X-Cache-Hits": "0", "Date": "Tue, 15 Mar 2016 04:37:05 GMT", "Age": "0", "X-Served-By": "cache-sjc3628-SJC", "Content-Type": "text/html", "Server": "lighttpd/1.4.28", "X-Status-Code": "404", "X-Cache": "MISS", "Content-Length": "345", "X-Timer": "S1458016625.375814,VS0,VE148", "Via": "1.1 varnish"}
```

### `xkcd.wbcr/1.response_body.dat`

* Contains the contents of the HTTP response body received from the origin HTTP server when obtaining HTTP resource #1.
* Binary file.


## Contributing

### Running the Tests

```
make test
```

### Gathering Code Coverage Metrics

```
make coverage
open htmlcov/index.html
```


## Known Limitations

* Sites that vary the content served at a particular URL depending on whether you are logged in can have only one version of the URL archived.



## License

This code is provided under the MIT License. See LICENSE file for details.
