# Simple Dockerfile for ngx_mruby

This is very simple Dockerfile and related files for [ngx_mruby](https://github.com/matsumoto-r/ngx_mruby).

## matsumotory/ngx-mruby image on Docker Hub
`matsumotory/ngx-mruby` image on Docker Hub is [a official ngx_mruby docker image](https://registry.hub.docker.com/u/matsumotory/ngx-mruby/). The image supports `ONBUILD` for below commands.

```
ONBUILD ADD docker/hook /usr/local/nginx/hook
ONBUILD ADD docker/conf /usr/local/nginx/conf
ONBUILD ADD docker/conf/nginx.conf /usr/local/nginx/conf/nginx.conf
```

So, you can create docker directory which was included nginx config files and ngx_mruby hook scripts into the same directory as Dockerfile before building Dockerfile.

## Simple Example
### prepare nginx config and ngx_mruby hook files
- `current_dir/Dockerfile`
```
FROM matsumotory/ngx-mruby:latest
MAINTAINER matsumotory
```
- `current_dir/docker/conf/nginx.conf`
```
user daemon;
daemon off;
master_process off;
worker_processes 1;
error_log stderr;

events {
    worker_connections  1024;
}

http {
    server {
        listen       80;

        location /mruby-hello {
            mruby_content_handler_code 'Nginx.echo "server ip: #{Nginx::Connection.new.local_ip}: hello ngx_mruby world."';
        }

        location /mruby-test {
            mruby_content_handler /usr/local/nginx/hook/test.rb;
        }
    }
}
```
- `current_dir/docker/hook/test.rb`
```
Nginx.echo "This is test for ngx_mruby"
```
### build and run
```
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
