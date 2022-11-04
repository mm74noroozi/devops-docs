# varnish docker config
## docker-compose
```yaml
version: "3.9"

services:
    varnish:
        image: hub.hamdocker.ir/library/varnish:6.6
        environment: 
            -   VARNISH_SIZE=2G
        tmpfs:
            -   /var/lib/varnish:exec
        volumes: 
            -   ./default.vcl:/etc/varnish/default.vcl
        ports: 
            -   8080:80
```
## config file
```
vcl 4.0;

backend default {
    .host = "prerender";
    .port = "3000";
}

sub vcl_recv {
    set req.http.host = "damirco.com";
    set req.url = regsub(req.url,"^","/http://nginx");
    unset req.http.Cache-Control;
    unset req.http.expires;
}

sub vcl_backend_response {
    unset beresp.http.set-cookie;
    unset beresp.http.Vary;
    set beresp.ttl = 200s;
    set beresp.grace = 20w;
}

sub vcl_deliver {
    if (obj.hits > 0) { # Add debug header to see if it's a HIT/MISS and the number of hits, disable when not needed
        set resp.http.X-Cache = "HIT";
    } else {
        set resp.http.X-Cache = "MISS";
    }
}
```
