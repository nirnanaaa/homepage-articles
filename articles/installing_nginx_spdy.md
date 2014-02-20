---
title: Installieren des SPDY2/3 Modules für NGINX auf Debian
date: 2014-02-20 19:14:00 GMT+02
---

SPDY ist ein Protokoll, welches von Google eingeführt wurde um die Ladezeiten von Websiten
enorm zu reduzieren. Es arbeitet Hand in Hand mit HTTPS damit ältere Browser ebenfalls bedient werden können.

SPDY3 benötigt folgende minimalen Browser Anforderungen:

* Firefox 27
* Google Chrome
* Opera 12.10
* Internet Explorer 11 auf Windows 8.1


Beim Internet Explorer 11 auf Windows 8.1 kann es vereinzelt auftreten dass beim ersten Laden der Fehler 
"Seite nicht gefunden" angezeigt wird. In diesem Falle einfach neu laden dann sollte es funktionieren.


### Installation

Als erstes installieren wir das normale Debian-NGINX-Paket wie folgt:

```sh
# apt-get install nginx
```


Anschließend müssen wir das neue Binary komplieren. Keine Angst bei meinen Installationen verlief dies stets einwandfrei.


Wir laden uns den Nginx Quellcode von der Offiziellen Seite: [[http://nginx.org/en/download.html]] . Man sollte darauf achten,
dass man Version `>= 1.5.10` verwendet, da in dieser erst der Standard 3.1 von SPDY eingeführt wurde.

```sh
wget http://nginx.org/download/nginx-1.5.10.tar.gz
tar xfvz nginx-1.5.10.tar.gz
cd nginx-1.5.10.tar.gz
```

Um die SPDY Funktionalität zu aktivieren benötigt man beim Konfigurieren den Schalter `--with-http_spdy_module`.

Eine Liste weiterer Konfigurationsoptionen findet man unter [[http://wiki.nginx.org/InstallOptions]] oder [[http://nginx.org/en/docs/configure.html]]

Zum Konfigurieren einfach den `./configure` Command ausführen:

```sh
./configure --prefix=/ \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--pid-path=/var/run/nginx.pid \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--user=www-data \
--group=www-data \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-pcre \
--with-http_realip_module \
--with-file-aio \
--with-http_spdy_module  \
--with-ipv6
```

Hier wird SPDY, SSL, IPv6, PCRE(RegExpressions), Gzip und einige weitere für mich nützliche Optionen aktiviert.

Die `--with-pcre` option erfordert zusätzlich das Paket `libpcre`, welches ganz einfach folgendermaßen installiert wird:

```sh
# apt-get install libpcre3 libpcre3-dev
```

Natürlich kann man hier auch selbst Anpassungen nach eigenem Ermessen vornehmen.
Wichtig hierbei ist nur dass das `--with-http_spdy_module` und das `--with-http_ssl_module` notwendig sind um SPDY
lauffähig zu bekommen.

Anschließend kann man das Ganze bauen:

```sh
make
```

Nun muss man das Binary an die Stelle des orginalen NGINX binaries kopieren:

```sh
cp objs/nginx `which nginx`
ln -s /etc/nginx /var/lib/nginx/conf
ln -s /var/log/nginx /var/lib/nginx/logs
```

Im Anschluss muss noch die `/etc/nginx/nginx.conf` angepasst werden:

```nginx
pid /var/run/nginx.pid;
lock_file /var/lock/nginx.lock;
```

Jetzt kann man den Server neustarten. Dies sollte ohne Fehler ablaufen:

```sh
# service nginx restart
```

### SPDY testen

Um SPDY zu testen benötigt man ein SSL Zertifikat. Dieses sollte nicht selbssigniert sein, ist aber trotzdem möglich.

Wenn man das SSL Zertifikat hat kann man die Konfiguration testen:

```sh
vi /etc/nginx/sites-enabled/spdytest
server {
    listen 443 ssl spdy;
    server_name your_domain_name.com;
    ssl on;
    ssl_certificate /path-to-your-cert.crt;
    ssl_certificate_key /path-to-your-key.key;
    root /tmp/spdy_test;
}
```

Anschließend erstellen wir den Document root + Test-Datei:


```sh
mkdir /tmp/spdy_test
echo it works > /tmp/spdy_test/index.html
```

Jetzt kann man per [[http://spdycheck.org/]] die Funktionalität testen.


## Wichtig bei NGINX Updates

Sollte es ein Update der Distribution geben, so muss das Binary erneut kopiert werden.






