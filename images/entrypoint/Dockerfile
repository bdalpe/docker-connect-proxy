# Build a custom version of NGINX with the proxy_connect module, copy it to the production container
FROM debian:bookworm-slim as builder

ENV NGINX_VERSION 1.25.4
ENV PROXY_CONNECT_MODULE_VERSION 0.0.5

WORKDIR /tmp
RUN apt-get update && apt-get install -y curl make patch gcc libpcre3-dev zlib1g-dev libssl-dev
RUN curl -L https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz | tar zxvf -
RUN curl -L https://github.com/chobits/ngx_http_proxy_connect_module/archive/refs/tags/v${PROXY_CONNECT_MODULE_VERSION}.tar.gz | tar zxvf -
WORKDIR /tmp/nginx-${NGINX_VERSION}
RUN patch -p1 < ../ngx_http_proxy_connect_module-${PROXY_CONNECT_MODULE_VERSION}/patch/proxy_connect_rewrite_102101.patch
RUN ./configure --add-module=../ngx_http_proxy_connect_module-${PROXY_CONNECT_MODULE_VERSION}/ --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=65532 --group=65532 --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_v3_module --with-cc-opt='-static -static-libgcc -O2 -ffile-prefix-map=/data/builder/debuild/nginx-${NGINX_VERSION}/debian/debuild-base/nginx-${NGINX_VERSION}=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-static'
RUN make && make install

# Grab lightweight bins from busybox for our entrypoint scripts
FROM busybox:1.36.1-uclibc as busybox
RUN ln -sf /bin/find /usr/bin/find  # NGINX entrypoint script has this hard-coded for some reason, so we can link it

# Copying some utilities from here
FROM nginx:1.25.4 as nginx
RUN sed -i 's,listen       80;,listen       8080;,' /etc/nginx/conf.d/default.conf \
    && sed -i '/user  nginx;/d' /etc/nginx/nginx.conf \
    && sed -i 's,/var/run/nginx.pid,/tmp/nginx.pid,' /etc/nginx/nginx.conf

# Note: nonroot tag = run as uid 65532 - see https://github.com/GoogleContainerTools/distroless/blob/main/base/base.bzl
FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=busybox /bin /bin/
COPY --from=busybox /usr/bin /usr/bin/
COPY --from=nginx /usr/bin/envsubst /usr/local/bin/envsubst
COPY --from=nginx --chown=65532:65532 /var/cache/nginx /var/cache/nginx
COPY --from=nginx --chown=65532:65532 /etc/nginx /etc/nginx/
COPY --from=nginx /var/log/nginx /var/log/nginx/
COPY --from=nginx /docker-entrypoint.sh /
COPY --from=nginx /docker-entrypoint.d /docker-entrypoint.d/
COPY --from=builder /sbin/nginx /sbin/nginx
COPY docker-entrypoint.d /docker-entrypoint.d/
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]