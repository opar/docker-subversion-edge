# CollabNet Subversion Edge - Container

This is a docker image of the Collabnet Subversion Edge Server

## Fork modifications

* Changed to run Apache HTTP SVN to run on port 80. Works better with Nginx.
* Some changes on bootstrap.sh and some files.
* Updated CSVN Edge version to 5.2.2.

## Usage

The image is exposing the data dir of csvn as a volume under `/opt/csvn/data`.
If you provide an empty host folder as volume the init scripts will take care of copying a basic configuration to the volume.
The container exposes the following ports:

 * 3343 - HTTP CSVN Admin Sites
 * 4434 - HTTPS CSVN Admin Sites (If SSL is enabled)
 * 80   - Apache HTTP SVN

The simplest way to start a subversion edge server is

    docker run -d danielasanome/subversion-edge

This will run the server. It will only be reachable from the docker host by using the container ip address

Exposing the ports from the host:
    
    docker run -d -p 10001:3343 -p 10002:4434 -p 10000:80 \
        --name svn-server danielasanome/subversion-edge

This will make the admin interface reachable under [http://docker-host:10001/csvn](http://docker-host:10001/csvn).

If you want to provide a host path for the data use command like this:
    
    docker run -d -p 10001:3343 -p 10002:4434 -p 10000:80 \
        -v /srv/svn-data:/opt/csvn/data --name svn-server danielasanome/subversion-edge
    

For Nginx configuration use like this:
```
    server {
      listen       80;
      server_name  dns.host-svn;
    
      location /csvn/ {  # CSVN Edge Admin Site
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_pass       http://docker-host:10001/csvn/;
      }
      location /viewvc/ {  # CSVN Edge SVN Explorer Site
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_pass       http://docker-host:10000/viewvc/;
      }
      location /viewvc-static/ {  # CSVN Edge SVN Explorer Site - Static files
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_pass       http://docker-host:10000/viewvc-static/;
      }
      location /svn {  # SVN Repository
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_pass       http://docker-host:10000/svn/;
      }
    }
```

This will make the admin interface reachable under [http://dns.host-svn/csvn/](http://dns.host-svn/csvn/) and SVN through [http://dns.host-svn/svn/](http://dns.host-svn/svn/).

For information to further configuration please consult the documentation at [CollabNet](http://collab.net/products/subversion).
