FROM openresty/openresty:1.15.8.2-6-alpine

RUN ["rm", "/etc/nginx/conf.d/default.conf"]

COPY --from=hairyhenderson/gomplate:v2.5.0-slim /gomplate /bin/gomplate

RUN apk add --no-cache perl curl && opm get bungle/lua-resty-template

COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY shellhub.conf /etc/nginx/conf.d/
COPY install.sh /usr/local/openresty/nginx/html/
COPY entrypoint.sh /

ENTRYPOINT ["/entrypoint.sh"]

CMD ["/usr/local/openresty/bin/openresty", "-g", "daemon off;"]
