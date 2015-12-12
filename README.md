docker-dns-proxy
================
app1.example.00:80 -> (dnsmasq, nginx) -> `app1.example` container in "00" custom docker 1.9 network

```
--- docker machine --------------------------
------ net: 00 ------------------------------
 [app1.example], [app2.example] ...
   ^
   | http://app1.example.00/ (e.g. 172.19.0.2)
   |
   |              /etc/hosts    --address=/.00/192.168.99.100
 [nginx] ------> [dnsmasq_in]  [dnsmaq_out]
- :80 ----------- :5353 ---------- :53 ------ expose port
   ^                                ^
   | http://app1.example.00/ (192.168.99.100)
   |                                |
---- localhost ------------------------------
 [browser] <---------> [/etc/resolver/docker]
```


Prerequisites
-------------
- docker 1.9

(I tested on OS X 10.10.5 with docker 1.9 installed by Kitematic.)


Assumptions
-----------
- A docker-machine running on `192.168.99.100`
- You use docker custom network named `00`
- Use `.00` as the TLD

Of course, you can alter those default if you wish.


Usage by example
----------------

### Let localhost resolve `.00` TLD for the docker machine
```
sudo bash -c 'echo "nameserver 192.168.99.100" > /etc/resolver/00'
```

### Create a custom docker network, `00`

```
$ docker network create 00
```

### Run an app container for test

Run python SimpleHTTPServer for example as `app1.example`.
```
$ docker run -it --rm --net 00 --name app1.example python:2 python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

### Build and run docker-dns-proxy

```
$ docker-compose build
```

```
$ docker-compose --x-networking up
Starting dockerdnsproxy_dnsmasq_out_1
Starting dockerdnsproxy_dnsmasq_in_1
Recreating dockerdnsproxy_nginx_1
Attaching to dockerdnsproxy_dnsmasq_out_1, dockerdnsproxy_dnsmasq_in_1, dockerdnsproxy_nginx_1
...
```

### Test!!

Enter `app1.example.00` in your browser or `curl http://app1.example.00`.

[![https://gyazo.com/75100cc61960cf330f50748ad47cf5a2](https://i.gyazo.com/75100cc61960cf330f50748ad47cf5a2.png)](https://gyazo.com/75100cc61960cf330f50748ad47cf5a2)

### Reload dnsmasq

If you restart the app container or run another app container, you may get `502 Bad Gateway`.
dnsmasq won't reload /etc/hosts automatically. You need let the dnsmasq process to reload /etc/hosts.

```
docker kill -s HUP dockerdnsproxy_dnsmasq_in_1
```
or
```
docker-compose restart dnsmasq
```
or simply shutdown and up docker-compose.

(I want a simple "watch /etc/hosts and HUP dnsmasq container if I can...)
