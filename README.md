docker-dns-proxy
================
app1.example.docker:80 -> (dnsmasq, nginx) -> app1 container in "example" custom docker 1.9 network

```
--- docker machine ------------------
------ net: example -----------------
 [app1] [app2]
   ^
   | http://app1.example/ (e.g. 172.19.0.2)
   |
 [nginx] ------> [dnsmasq]
- :80 ----------- :53 --------------
   ^               ^
   | http://app1.example.docker/ (192.168.99.100)
   |               |
---- localhost ----------------------
 [browser] <-> [/etc/resolver/docker]
```


Prerequisites
-------------
- docker 1.9

(I tested on OS X 10.10.5 with docker 1.9 installed by Kitematic.)


Assumptions
-----------
- A docker-machine running on `192.168.99.100`
- You use docker custom network named `example`
- Use `.docker` as TLD

Of course, you can alter those default if you wish.


Usage by example
----------------

### Let localhost resolve `docker` TLD on the docker-machine
```
sudo bash -c 'echo "nameserver 192.168.99.100" > /etc/resolver/docker'
```

### Create a custom docker network

```
$ docker network create example
```

### Run an app container

Run python SimpleHTTPServer for example as `app1`.
```
$ docker run -it --rm --net example --name app1 python:2 python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

### Build and run docker-dns-proxy

```
$ docker-compose build
```

```
$ docker-compose --x-networking up
Starting dockerdnsproxy_nginx_1
Creating dockerdnsproxy_dnsmasq_1
Attaching to dockerdnsproxy_nginx_1, dockerdnsproxy_dnsmasq_1
nginx_1   | 2015/12/06 02:21:06 [notice] 1#0: using the "epoll" event method
nginx_1   | 2015/12/06 02:21:06 [notice] 1#0: nginx/1.6.2
nginx_1   | 2015/12/06 02:21:06 [notice] 1#0: OS: Linux 4.1.12-boot2docker
nginx_1   | 2015/12/06 02:21:06 [notice] 1#0: getrlimit(RLIMIT_NOFILE): 1048576:1048576
nginx_1   | 2015/12/06 02:21:06 [notice] 1#0: start worker processes
nginx_1   | 2015/12/06 02:21:06 [notice] 1#0: start worker process 6
dnsmasq_1 | dnsmasq: started, version 2.72 cachesize 150
dnsmasq_1 | dnsmasq: compile time options: IPv6 GNU-getopt DBus i18n IDN DHCP DHCPv6 no-Lua TFTP conntrack ipset auth DNSSEC loop-detect
dnsmasq_1 | dnsmasq: reading /etc/resolv.conf
dnsmasq_1 | dnsmasq: using nameserver 10.0.1.1#53
dnsmasq_1 | dnsmasq: read /etc/hosts - 9 addresses
```

### Test!!

Enter `app1.example.docker` in your browser or `curl http://app1.example.docker`.

[![https://gyazo.com/a7bc10db80f027a2441d3d63b41161da](https://i.gyazo.com/a7bc10db80f027a2441d3d63b41161da.png)](https://gyazo.com/a7bc10db80f027a2441d3d63b41161da)

### Reload dnsmasq

If you restart the app container or run another app container, you may get `502 Bad Gateway`.
dnsmasq won't reload /etc/hosts automatically. You need let the dnsmasq process to reload /etc/hosts.

```
docker kill -s HUP dockerdnsproxy_dnsmasq_1
```
,
```
docker-compose restart dnsmasq
```
or simply shutdown and up docker-compose.

(I want a simple "watch /etc/hosts and HUP dnsmasq container if I can...)
