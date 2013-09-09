nginx
=====

my nginx config for owncloud


#OwnCloud
`
server {
 server_name servername;
 return 301 https://$server_name$request_uri;  # enforce https
}
`
# owncloud (ssl/tls)
server {
  listen 443 ssl;
  ssl_certificate certificate;
  ssl_certificate_key certificate
  server_name servername
  root /var/www/vhosts/owncloud;
  index index.php;
  client_max_body_size 1000M; # set maximum upload size

  # deny direct access
  location ~ ^/(data|config|\.ht|db_structure\.xml|README) {
    deny all;
  }

  # default try order
  location / {
    try_files $uri $uri/ @webdav;
  }

  # owncloud WebDAV
  location @webdav {
    fastcgi_split_path_info ^(.+\.php)(/.*)$;
    fastcgi_pass 127.0.0.1:9000; # or use php-fpm with: "unix:/var/run/php-fpm/php-fpm.sock;"
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param HTTPS on;
    include fastcgi_params;
  }

  # enable php
  location ~ \.php$ {
    try_files $uri = 404;
    fastcgi_pass 127.0.0.1:9000; # or use php-fpm with: "unix:/var/run/php-fpm/php-fpm.sock;"
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param HTTPS on;
    include fastcgi_params;
  }
}
