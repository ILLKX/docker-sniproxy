# docker-sniproxy

docker-sniproxy is a SNI Proxy boxed in a Docker image built by [Tommy Lau](http://tommy.net.cn/).

## What is SNI Proxy

[SNI Proxy](https://github.com/dlundquist/sniproxy) proxies incoming HTTP and TLS connections based on the hostname contained in the initial request of the TCP session. This enables HTTPS name-based virtual hosting to separate backend servers without installing the private key on the proxy machine.

## What's included?

The latest SNI Proxy from official release and nothing more.

## How to use this image

Get the docker image by running the following commands:

``` bash
$ docker pull tommylau/sniproxy
$ docker run --name sniproxy ---net=host -v /path/to/sniproxy:/etc/sniproxy -d tommylau/sniproxy
```

Example config file

```
# sniproxy example configuration file
# lines that start with # are comments
# lines with only white space are ignored

user daemon

# PID file
pidfile /var/run/sniproxy.pid

error_log {
    # Log to the daemon syslog facility
    syslog deamon

    # Alternatively we could log to file
    #filename /var/log/sniproxy/sniproxy.log

    # Control the verbosity of the log
    priority notice
}

# blocks are delimited with {...}
listen 80 {
    proto http
    table http_hosts
    # Fallback backend server to use if we can not parse the client request
    fallback localhost:8080

    access_log {
        filename /var/log/sniproxy/http_access.log
        priority notice
    }
}

listen 443 {
    proto tls
    table https_hosts

    access_log {
        filename /var/log/sniproxy/https_access.log
        priority notice
    }
}

# named tables are defined with the table directive
table http_hosts {
    example.com 192.0.2.10:8001
    example.net 192.0.2.10:8002
    example.org 192.0.2.10:8003

# pattern:
# 	valid Perl-compatible Regular Expression that matches the
# 	hostname
#
# target:
#	- a DNS name
#	- an IP address (with optional port)
#	- '*' to use the hostname that the client requested
#
# pattern	target
#.*\.itunes\.apple\.com$	*:443
#.*	127.0.0.1:4443
}

# named tables are defined with the table directive
table https_hosts {
    # When proxying to local sockets you should use different tables since the
    # local socket server most likely will not autodetect which protocol is
    # being used
    example.org unix:/var/run/server.sock
}

# if no table specified the default 'default' table is defined
table {
    # if no port is specified default HTTP (80) and HTTPS (443) ports are
    # assumed based on the protocol of the listen block using this table
    example.com 192.0.2.10
    example.net 192.0.2.20
}
```

