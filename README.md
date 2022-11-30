# Simple Dockerfile for ngx_mruby

This is very simple Dockerfile and related files for [ngx_mruby](https://github.com/matsumoto-r/ngx_mruby).

Connecting to https://hub.docker.com/r/matsumotory/docker-ngx_mruby/.

## matsumotory/ngx-mruby image on Docker Hub
`matsumotory/ngx-mruby` image on Docker Hub is [an official ngx_mruby docker image](https://registry.hub.docker.com/u/matsumotory/ngx-mruby/). This image supports `ONBUILD` for below commands.

```
ONBUILD ADD docker/hook /usr/local/nginx/hook
ONBUILD ADD docker/conf /usr/local/nginx/conf
ONBUILD ADD docker/conf/nginx.conf /usr/local/nginx/conf/nginx.conf
```

So, you can create `docker/` directory which include the nginx config files (`docker/conf/`) and ngx_mruby hook scripts (`docker/hook/`) into the same directory as Dockerfile before building Dockerfile.

### nginx version and modules on matsumotory/ngx-mruby image
We built nginx with ngx_mruby linked with commonly-used some nginx modules.
```
$ sudo docker run -p 80:80 matsumotory/ngx-mruby /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.7.10
built by gcc 4.8.2 (Ubuntu 4.8.2-19ubuntu1)
TLS SNI support enabled
configure arguments: --add-module=/usr/local/src/ngx_mruby --add-module=/usr/local/src/ngx_mruby/dependence/ngx_devel_kit --with-http_stub_status_module --with-http_ssl_module --prefix=/usr/local/nginx --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module
```

## Simple Example
### prepare nginx config and ngx_mruby hook files
- `current_dir/Dockerfile`
```
FROM matsumotory/ngx_mruby:master
MAINTAINER matsumotory
```
- `current_dir/docker/conf/nginx.conf`
```nginx
user daemon;
daemon off;
master_process off;
worker_processes 1;
error_log stderr;

events {
    worker_connections 1024;
}

http {
    server {
        listen 80;

        location /mruby-hello {
            mruby_content_handler_code 'Nginx.echo "server ip: #{Nginx::Connection.new.local_ip}: hello ngx_mruby world."';
        }

        location /mruby-test {
            mruby_content_handler /usr/local/nginx/hook/test.rb;
        }

        location / {
            resolver 8.8.8.8;
            mruby_set_code $backend '["blog.matsumoto-r.jp", "hb.matsumoto-r.jp"][rand(2)]';
            proxy_pass http://$backend;
        }
    }
}
```
- `current_dir/docker/hook/test.rb`
```ruby
Nginx.echo "This is test for ngx_mruby"
```
### build and run
```bash
cd current_dir/
sudo docker build -t local/docker-ngx_mruby .
sudo docker run -p 80:80 -t local/docker-ngx_mruby
```
### access to ngx_mruby
```
$ curl http://127.0.0.1/mruby-hello
server ip: 172.17.0.200: hello ngx_mruby world.
$ curl http://127.0.0.1/mruby-test
This is test for ngx_mruby
```

### enjoy!!!

# License
under [the MIT License](/MITL):

* http://www.opensource.org/licenses/mit-license.php

