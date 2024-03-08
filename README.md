# NGINX CONNECT Proxy

This is a container image of NGINX compiled from source with the [ngx_http_proxy_connect_module](https://github.com/chobits/ngx_http_proxy_connect_module) added.

The image uses [Distroless `static-debian12:nonroot`](https://github.com/GoogleContainerTools/distroless) as the base image and is roughly 16 MB in size.

## Image Variants

See the [`images` folder](images) for Dockerfile specs. Images are published via the GitHub Packages for this repo.

### Static
Contains the NGINX binary and sets ownership on required folders.

### Entrypoint

This image inherits the `docker-entrypoint.d` script functionality from the [`nginx` image](https://github.com/nginxinc/docker-nginx). Binaries from the `busybox` image have been imported so basic shell commands work.

## License

Copyright 2024 Brendan Dalpe

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.