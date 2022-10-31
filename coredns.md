# Coredns
## zone file (zones.cfg)
```
emofid.com {
    forward . 10.1.8.11
    log
    errors
    cache
}
mofid.dev {
    forward . 10.1.8.11
    log
    errors
    cache
}
mofid.dc {
    forward . 10.1.8.11
    log
    errors
    cache
}

cluster.local {
    file cluster.local.txt
    log
    errors
    cache
}

. {
    forward . tls://178.22.122.100 {
       tls_servername free.shecan.ir
       health_check 5s
    }
    log
    errors
    cache
}
```
## domain record ?? SOA
```
@	3600 IN	SOA sns.dns.icann.org. noc.dns.icann.org. (
				2017042745 ; serial
				7200       ; refresh (2 hours)
				3600       ; retry (1 hour)
				1209600    ; expire (2 weeks)
				3600       ; minimum (1 hour)
				)

	3600 IN NS a.iana-servers.net.
	3600 IN NS b.iana-servers.net.
* 60  IN A  127.0.0.1
```
## run app
```console
conredns --conf zones.cfg
```
