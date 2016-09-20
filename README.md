# HAProxy Content Pack for Graylog (https://www.graylog.org/)

Tested with HAProxy 1.5/rsyslog/Graylog 2

This content pack provides example configuration and useful dashboard for Haproxy load balancer :
* Top hourly clients
* Top hourly backends
* Backends with retries > 0 in 5 days
* Frontend connections in 7 days

## Includes

* Haproxy sample configuration (in order to format log in JSON)
* Rsyslog configuration for catching JSON logs of chrooted Haproxy and transfer to Graylog (change with your graylog server or LB)
* A sample dashboard -> feel free to adapt this !!
* No stream in this content pack (if you want to make one just create it with rule: application_name:haproxy)

## Requirements

* Haproxy 1.5
* Graylog 2 (or 2.1)
* Rsyslog collecting logs, other log collectors will work but may require modifying the searches to match the different fields outputted by other collectors

## /etc/haproxy/haproxy.cfg Example
```
global
    log         127.0.0.1 local2
    log-send-hostname

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     30000
    user        haproxy
    group       haproxy
    daemon
...
listen myapp
    mode tcp
    option tcplog
    log-format {"haproxy_clientIP":"%ci","haproxy_clientPort":"%cp","haproxy_dateTime":"%t","haproxy_frontendNameTransport":"%ft","haproxy_backend":"%b","haproxy_serverName":"%s","haproxy_Tw":"%Tw","haproxy_Tc":"%Tc","haproxy_Tt":"%Tt","haproxy_bytesRead":"%B","haproxy_terminationState":"%ts","haproxy_actconn":%ac,"haproxy_FrontendCurrentConn":%fc,"haproxy_backendCurrentConn":%bc,"haproxy_serverConcurrentConn":%sc,"haproxy_retries":%rc,"haproxy_srvQueue":%sq,"haproxy_backendQueue":%bq,"haproxy_backendSourceIP":"%bi","haproxy_backendSourcePort":"%bp"}
    option logasap
    balance leastconn
	....

```


## Rsyslog sample configuration /etc/rsyslog.d/rsyslog-haproxy.conf (dont forget to change the IP address !)
```
# Centralisation des logs vers Graylog
# Fichier specifique pour HAProxy

# HAProxy est chroote donc subtilite ici oblige de renvoyer les logs sur une
# entree udp 514. HAProxy log en local2. Rsyslog envoie ensuite tout ca dans
# un fichier de log pour le statut et un fichier pour le suivi
# Ce fichier de suivi sera formate en JSON et transmis à Graylog

$template GRAYLOGRFC5424,"<%PRI%>%PROTOCOL-VERSION% %TIMESTAMP:::date-rfc3339% %HOSTNAME% %APP-NAME% %PROCID% %MSGID% %STRUCTURED-DATA% %msg%\n"

# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514
$UDPServerAddress 127.0.0.1

local2.=info            /var/log/haproxy.log;GRAYLOGRFC5424
local2.notice           /var/log/haproxy-status.log;GRAYLOGRFC5424

if $syslogtag contains 'haproxy' and $msg contains 'stats' then ~
if $syslogtag contains 'haproxy' then @@xxx.xxx.xxx.xxx:12211;GRAYLOGRFC5424
:syslogtag, contains, "haproxy" ~

```

## Screenshots

![Screenshot](/screenshot.png?raw=true "Dashboard Screenshot")
