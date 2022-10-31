# nginx serve static + proxy
## docker-compose.yml
```yaml
version: "3.9"

services:
  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - 80:80
    volumes:
      - ./assets:/usr/share/nginx/html/
      - ./nginx.conf:/etc/nginx/conf.d/default.conf  

  sms_marketing:
    build: . 
    restart: always
    volumes:
      - ./assets:/code/assets/
      - ./db.sqlite3:/code/db.sqlite3

```
## nginx.conf
```nginx
server {
    listen 80 default_server;
    server_name example.com;
    location ~ ^/(static|media) {
        root   /usr/share/nginx/html;
    }

    location / {
        proxy_pass_header  Set-Cookie;
        proxy_set_header   Host               $host;
        proxy_set_header   X-Real-IP          $remote_addr;
        proxy_set_header   X-Forwarded-Proto  $scheme;
        proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_pass   http://sms_marketing:8000;
    }

}
```
## total config
```nginx
proxy_cache_path /var/cache/nginx-static levels=1:2 keys_zone=static:10m inactive=120d;
proxy_cache_path /var/cache/nginx-zerodown levels=1:2 keys_zone=zerodown:10m inactive=120d;

server
{
	listen 80;
	server_name homeca.ir www.homeca.ir homeca.co www.homeca.co lab-test.drnext.ir;

	access_log /var/log/nginx/homeca.access.log main_ext;
	error_log /var/log/nginx/homeca.error.log warn;

	location ~ ^/mag
	{
		client_max_body_size 100M;
		proxy_pass http://mag.homeca-site.svc.cluster.local:80;
		proxy_pass_header Set-Cookie;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-Proto 'https';
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}

	location ~ ^/client/dashboard/v2/
	{
		client_max_body_size 10M;
		proxy_pass http://client-pwa-production.client-pwa.svc.cluster.local:3000;
		proxy_pass_header Set-Cookie;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-Proto 'https';
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}

	location ~ ^/(services|articles|aboutus|contactus|lab)/
	{
		proxy_cache zerodown;
		proxy_cache_methods GET;
		proxy_cache_min_uses 2;
		proxy_cache_valid 200 301 21m;

		proxy_hide_header Set-Cookie;
		proxy_ignore_headers Set-Cookie Cache-Control Vary;
		# add_header Set-Cookie \"\";
		add_header X-HM-Cache-Status $upstream_cache_status;
		proxy_cache_key $scheme$request_uri$args$cookie_sessionid;
		proxy_cache_background_update on;
		proxy_cache_use_stale updating;

		client_max_body_size 10M;
		proxy_pass http://homeca-django-production.homeca-site.svc.cluster.local:8000;
		# proxy_pass_header  Set-Cookie;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-Proto 'https';
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
	location /
	{
		client_max_body_size 10M;
		proxy_pass http://homeca-django-production.homeca-site.svc.cluster.local:8000;
		proxy_pass_header Set-Cookie;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_read_timeout 300s;
		proxy_connect_timeout 75s;
		proxy_set_header X-Forwarded-Proto 'https';
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
	location /admin
	{
		client_max_body_size 100M;
		proxy_pass http://homeca-django-production.homeca-site.svc.cluster.local:8000;
		proxy_pass_header Set-Cookie;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-Proto 'https';
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
	location ~ ^/(static|media)/
	{
		proxy_cache static;
		proxy_cache_background_update on;
		add_header X-HM-Cache-Status $upstream_cache_status;
		proxy_ignore_headers Set-Cookie Cache-Control Vary;
		proxy_hide_header Vary;
		proxy_hide_header Access-Control-Allow-Origin;
		add_header Access-Control-Allow-Origin \"*\";
		proxy_cache_key $host$request_uri;
		proxy_cache_min_uses 2;
		proxy_cache_valid 200 10h;

		client_max_body_size 100M;
		proxy_pass http://minio-1.homeca-site.svc.cluster.local:9000;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-Proto 'https';
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}

server
{
	listen 80;
	server_name static.homeca.ir;
	access_log /var/log/nginx/homeca-static.access.log main_ext;
	error_log /var/log/nginx/homeca-static.error.log;

	proxy_cache static;
	proxy_cache_background_update on;
	add_header X-HM-Cache-Status $upstream_cache_status;
	proxy_ignore_headers Set-Cookie Cache-Control Vary;
	proxy_hide_header Vary;
	proxy_hide_header Access-Control-Allow-Origin;
	add_header Access-Control-Allow-Origin \"*\";
	proxy_cache_key $host$request_uri;
	proxy_cache_min_uses 2;
	proxy_cache_valid 200 10h;

	location / {
		client_max_body_size 100M;
		proxy_pass http://minio-1.homeca-site.svc.cluster.local:9000;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-Proto 'https';
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
server
{
	listen 80 ;
	server_name static-console.homeca.ir;
	location / {
		proxy_pass http://minio-1.homeca-site.svc.cluster.local:9001;
	}
}
server
{
	listen 80 ;
	location / {
		proxy_pass http://homeca-django-production.homeca-site.svc.cluster.local:8000;
	}
}
```
