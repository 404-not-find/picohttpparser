PicoHTTPParser
=============

Copyright (c) 2009-2014 [Kazuho Oku](https://github.com/kazuho), [Tokuhiro Matsuno](https://github.com/tokuhirom), [Daisuke Murase](https://github.com/typester), [Shigeo Mitsunari](https://github.com/herumi)

PicoHTTPParser is a tiny, primitive, fast HTTP request/response parser.

Unlike most parsers, it is stateless and does not allocate memory by itself.
All it does is accept pointer to buffer and the output structure, and setups the pointers in the latter to point at the necessary portions of the buffer.

The code is widely deployed within Perl applications through popular modules that use it, including [Plack](https://metacpan.org/pod/Plack), [Starman](https://metacpan.org/pod/Starman), [Starlet](https://metacpan.org/pod/Starlet), [Furl](https://metacpan.org/pod/Furl).  It is also the HTTP/1 parser of [H2O](https://github.com/h2o/h2o).

Check out [test.c] to find out how to use the parser.

The software is dual-licensed under the Perl License or the MIT License.

Usage
-----

The library exposes four functions: `phr_parse_request`, `phr_parse_response`, `phr_parse_headers`, `phr_decode_chunked`.

The example below reads an HTTP request from socket `sock` using `read(2)`, parses it using `phr_parse_request`, and prints the details.

```
char buf[4096], *method, *path;
int pret, minor_version;
struct phr_header headers[100];
size_t buflen = 0, prevbuflen = 0, method_len, path_len, num_headers;
ssize_t rret;

while (1) {
    /* read the request */
    while ((rret = read(sock, buf + len, sizeof(buf) - len)) == -1 && errno == EINTR)
        ;
    if (rret <= 0)
        return IOError;
    len += rret;
    /* parse the request */
    num_headers = sizeof(headers) / sizeof(headers[0]);
    pret = phr_parse_request(rbuf, rlen, &method, &method_len, &path, &path_len,
                             &minor_version, &headers, &num_headers, prevbuflen);
    if (pret > 0)
        break; /* successfully parsed the request */
    else if (pret == -1)
        return ParseError;
    /* request is incomplete, continue the loop */
    assert(pret == -2);
    if (buflen == sizeof(buf))
        return RequestIsTooLongError;
    prevbuflen = buflen;
}

printf("request is %d bytes long\n", rlen);
printf("method is %.*s\n", (int)method_len, method);
printf("path is %.*s\n", (int)path_len, path);
printf("HTTP version is 1.%d\n", minor_version);
printf("headers:\n");
for (i = 0; i != num_headers; ++i)
  printf("%.*s: %.*s\n", (int)headers[i].name_len, headers[i].name,
         (int)headers[i].value_len, headers[i].value);
```

`phr_parse_response` and `phr_parse_headers` provide similar interfaces as `phr_parse_request`.  `phr_parse_response` parses an HTTP response, and `phr_parse_headers` parses the headers only.

Benchmark
---------

![benchmark results](http://i.gyazo.com/a85c18d3162dfb46b485bb41e0ad443a.png)

The benchmark code is from [fukamachi/fast-http@6b91103](https://github.com/fukamachi/fast-http/tree/6b9110347c7a3407310c08979aefd65078518478).

The internals of picohttpparser has been described to some extent in [my blog entry]( http://blog.kazuhooku.com/2014/11/the-internals-h2o-or-how-to-write-fast.html).
