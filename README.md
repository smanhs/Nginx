Steps to follow
Run docker-compose up
docker-compose up
Go to localhost:8080/

This delivers the main app

Go to localhost:8080/app/

Reload the app and it redirects to app1 and app2
==================================================
advanced-nginx-cheatsheet
Work In Progress, more explanations will be added soon

Table of content

advanced-nginx-cheatsheet
Nginx Performance
Load-Balancing
php-fpm Unix socket
php-fpm TCP
HTTP load-balancing
HTTP/3 QUIC support
WordPress Fastcgi cache
mapping fastcgi_cache_bypass conditions
Define fastcgi_cache settings
fastcgi_cache vhost example
Nginx as a Proxy
Simple Proxy
Proxy in a subfolder
Proxy keepalive for websocket
Reverse-Proxy For Apache
Nginx Security
Denying access
common backup and archives files
Deny access to hidden files & directory
Blocking common attacks
base64 encoded url
javascript eval() url
Nginx SEO
robots.txt location
Make a website not indexable
Nginx Media
MP4 stream module
WebP images
Nginx Performance
Load-Balancing
php-fpm Unix socket
upstream php {
    least_conn;

    server unix:/var/run/php/php-fpm.sock;
    server unix:/var/run/php/php-two-fpm.sock;

    keepalive 5;
}
php-fpm TCP
upstream php {
    least_conn;

    server 127.0.0.1:9090;
    server 127.0.0.1:9091;

    keepalive 5;
}
HTTP load-balancing
# Upstreams
upstream backend {
    least_conn;

    server 10.10.10.1:80;
    server 10.10.10.2:80;
}

server {

    server_name site.ltd;

    location / {
        proxy_pass http://backend;
        proxy_redirect      off;
        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
HTTP/3 QUIC support
HTTP/3 QUIC support is available in Nginx from 1.25.5 release and earlier. An SSL library that provides QUIC support such as BoringSSL, LibreSSL, or QuicTLS is recommended to build and run this module. Otherwise, when using the OpenSSL library, OpenSSL compatibility layer will be used that does not support early data.

reuseport directive is only available on a single vhost.
If Nginx isn't built with more_set_headers module, you can use add_header X-protocol $server_protocol always; and add_header Alt-Svc 'h3=":$server_port"; ma=86400';
# Main virtualhost
server {

    server_name yoursite.tld;

    # enable http/2
    http2 on;

    # display http version used in header (optional)
    more_set_headers "X-protocol : $server_protocol always";

    # Advertise HTTP/3 QUIC support (required)
    more_set_headers 'Alt-Svc h3=":$server_port"; ma=86400';

    # enable [QUIC address validation](https://datatracker.ietf.org/doc/html/rfc9000#name-address-validation)
    quic_retry on;

    # Listen on port 443 with HTTP/3 QUIC as default_server
    listen 443 quic reuseport default_server;
    listen [::]:443 quic reuseport default_server;

    # listen on port 443 with HTTP/2
    listen 443 ssl;
    listen [::]:443 ssl;

    # enable HSTS with HSTS preloading
    more_set_headers "Strict-Transport-Security : max-age=31536000; includeSubDomains; preload";

    # SSL certificate
    ssl_certificate /etc/letsencrypt/live/yoursite.tld/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/yoursite.tld/key.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/yoursite.tld/ca.pem;

    # enable SSL Stapling
    ssl_stapling_verify on;
}
# Other virtualhosts
server {

    server_name othersite.tld;

    # enable http/2
    http2 on;

    # display http version used in header (optional)
    more_set_headers "X-protocol : $server_protocol always";

    # Advertise HTTP/3 QUIC support (required)
    more_set_headers 'Alt-Svc h3=":$server_port"; ma=86400';

    # enable [QUIC address validation](https://datatracker.ietf.org/doc/html/rfc9000#name-address-validation)
    quic_retry on;

    # Listen on port 443 with HTTP/3 QUIC
    listen 443 quic;
    listen [::]:443 quic;

    # listen on port 443 with HTTP/2
    listen 443 ssl;
    listen [::]:443 ssl;

    # enable HSTS with HSTS preloading
    more_set_headers "Strict-Transport-Security : max-age=31536000; includeSubDomains; preload";

    # SSL certificate
    ssl_certificate /etc/letsencrypt/live/othersite.tld/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/othersite.tld/key.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/othersite.tld/ca.pem;

    # enable SSL Stapling
    ssl_stapling_verify on;
}
WordPress Fastcgi cache
mapping fastcgi_cache_bypass conditions
To put inside a configuration file in /etc/nginx/conf.d/

# do not cache xmlhttp requests
map $http_x_requested_with $http_request_no_cache {
    default 0;
    XMLHttpRequest 1;
}
# do not cache requests for the following cookies
map $http_cookie $cookie_no_cache {
    default 0;
    "~*wordpress_[a-f0-9]+" 1;
    "~*wp-postpass" 1;
    "~*wordpress_logged_in" 1;
    "~*wordpress_no_cache" 1;
    "~*comment_author" 1;
    "~*woocommerce_items_in_cart" 1;
    "~*woocommerce_cart_hash" 1;
    "~*wptouch_switch_toogle" 1;
    "~*comment_author_email_" 1;
}
# do not cache requests for the following uri
map $request_uri $uri_no_cache {
    default 0;
    "~*/wp-admin/" 1;
    "~*/wp-[a-zA-Z0-9-]+.php" 1;
    "~*/feed/" 1;
    "~*/index.php" 1;
    "~*/[a-z0-9_-]+-sitemap([0-9]+)?.xml" 1;
    "~*/sitemap(_index)?.xml" 1;
    "~*/wp-comments-popup.php" 1;
    "~*/wp-links-opml.php" 1;
    "~*/wp-.*.php" 1;
    "~*/xmlrpc.php" 1;
}
# do not cache request with args (like site.tld/index.php?id=1)
map $query_string $query_no_cache {
    default 1;
    "" 0;
}
# map previous conditions with the variable $skip_cache
map $http_request_no_cache$cookie_no_cache$uri_no_cache$query_no_cache $skip_cache {
    default 1;
    0000 0;
}
Define fastcgi_cache settings
To put inside another configuration file in /etc/nginx/conf.d

# FastCGI cache settings
fastcgi_cache_path /var/run/nginx-cache levels=1:2 keys_zone=WORDPRESS:360m inactive=24h max_size=256M;
fastcgi_cache_key "$scheme$request_method$host$request_uri";
fastcgi_cache_use_stale error timeout invalid_header updating http_500 http_503;
fastcgi_cache_lock on;
fastcgi_cache_lock_age 5s;
fastcgi_cache_lock_timeout 5s;
fastcgi_cache_methods GET HEAD;
fastcgi_cache_background_update on;
fastcgi_cache_valid 200 24h;
fastcgi_cache_valid 301 302 30m;
fastcgi_cache_valid 499 502 503 1m;
fastcgi_cache_valid 404 1h;
fastcgi_cache_valid any 1h;
fastcgi_buffers 16 16k;
fastcgi_buffer_size 32k;
fastcgi_param SERVER_NAME $http_host;
fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
fastcgi_keep_conn on;
# only available with Nginx 1.15.6 and earlier
fastcgi_socket_keepalive on;
To work with cookies, you can edit the fastcgi_cache_key. Cookie can be added with variable $cookie_{COOKIE_NAME}. For example, the WordPress plugin Polylang use a cookie named pll_language, so the directive fastcgi_cache_key would be :

fastcgi_cache_key "$scheme$request_method$host$request_uri$cookie_pll_language";
fastcgi_cache vhost example
server {

    server_name domain.tld;

    access_log /var/log/nginx/domain.tld.access.log;
    error_log /var/log/nginx/domain.tld.error.log;

    root /var/www/domain.tld/htdocs;
    index index.php index.html index.htm;

    # add X-fastcgi-cache header to see if requests are cached
    add_header X-fastcgi-cache $upstream_cache_status;

    # default try_files directive for WP 5.0+ with pretty URLs
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }
    # pass requests to fastcgi with fastcgi_cache enabled
    location ~ \.php$ {
        try_files $uri =404;
        include fastcgi_params;
        fastcgi_pass php;
        fastcgi_cache_bypass $skip_cache;
        fastcgi_no_cache $skip_cache;
        fastcgi_cache WORDPRESS;
        fastcgi_cache_valid 200 30m;
    }
    # block to purge nginx cache with nginx was built with ngx_cache_purge module
    location ~ /purge(/.*) {
        fastcgi_cache_purge WORDPRESS "$scheme$request_method$host$1";
        access_log off;
    }

}
Nginx as a Proxy
Simple Proxy
location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_redirect      off;
        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
Proxy in a subfolder
location /folder/ { # The / is important!
        proxy_pass http://127.0.0.1:3000/;# The / is important!
        proxy_redirect      off;
        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
Proxy keepalive for websocket
# Upstreams
upstream backend {
    server 127.0.0.1:3000;
    keepalive 5;
}
# HTTP Server
server {
    server_name site.tld;
    error_log /var/log/nginx/site.tld.access.log;
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forward-Proto http;
        proxy_set_header X-Nginx-Proxy true;
        proxy_redirect off;
    }
}
Reverse-Proxy For Apache
server {

    server_name domain.tld;

    access_log /var/log/nginx/domain.tld.access.log;
    error_log /var/log/nginx/domain.tld.error.log;

    root /var/www/domain.tld/htdocs;

    # pass requests to Apache backend
    location / {
        proxy_pass http://backend;
    }
    # handle static files with a fallback
    location ~* \.(ogg|ogv|svg|svgz|eot|otf|woff|woff2|ttf|m4a|mp4|ttf|rss|atom|jpe?g|gif|cur|heic|png|tiff|ico|zip|webm|mp3|aac|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf|swf|webp)$ {
        add_header "Access-Control-Allow-Origin" "*";
        access_log off;
        log_not_found off;
        expires max;
        try_files $uri @fallback;
    }
    # fallback to pass requests to Apache if files are not found
    location @fallback {
        proxy_pass http://backend;
    }
}
Nginx Security
Denying access
common backup and archives files
location ~* "\.(old|orig|original|php#|php~|php_bak|save|swo|aspx?|tpl|sh|bash|bak?|cfg|cgi|dll|exe|git|hg|ini|jsp|log|mdb|out|sql|svn|swp|tar|rdf)$" {
    deny all;
}
Deny access to hidden files & directory
location ~ /\.(?!well-known\/) {
    deny all;
}
Blocking common attacks
base64 encoded url
location ~* "(base64_encode)(.*)(\()" {
    deny all;
}
javascript eval() url
location ~* "(eval\()" {
    deny all;
}
Nginx SEO
robots.txt location
location = /robots.txt {
# Some WordPress plugin gererate robots.txt file
# Refer #340 issue
    try_files $uri $uri/ /index.php?$args @robots;
    access_log off;
    log_not_found off;
}
location @robots {
    return 200 "User-agent: *\nDisallow: /wp-admin/\nAllow: /wp-admin/admin-ajax.php\n";
}
Make a website not indexable
add_header X-Robots-Tag "noindex";

location = /robots.txt {
  return 200 "User-agent: *\nDisallow: /\n";
}
Nginx Media
MP4 stream module
location /videos {
    location ~ \.(mp4)$ {
        mp4;
        mp4_buffer_size       1m;
        mp4_max_buffer_size   5m;
        add_header Vary "Accept-Encoding";
        add_header "Access-Control-Allow-Origin" "*";
        add_header Cache-Control "public, no-transform";
        access_log off;
        log_not_found off;
        expires max;
    }
}
WebP images
Mapping conditions to display WebP images

# serve WebP images if web browser support WebP
map $http_accept $webp_suffix {
   default "";
   "~*webp" ".webp";
}
Set conditional try_files to server WebP image :

if web browser support WebP
if WebP alternative exist
# webp rewrite rules for jpg and png images
# try to load alternative image.png.webp before image.png
location /wp-content/uploads {
    location ~ \.(png|jpe?g)$ {
        add_header Vary "Accept-Encoding";
        add_header "Access-Control-Allow-Origin" "*";
        add_header Cache-Control "public, no-transform";
        access_log off;
        log_not_found off;
        expires max;
        try_files $uri$webp_suffix $uri =404;
    }
}

