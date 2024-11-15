# Examen DNS - Anxo Vázquez

Crea un repositorio para contestar todo o exame.
Este repositorio ten que conter tódo-los ficheiros necesarios para xustifica-la túas respostas.

Contesta as seguintes preguntas, xustificándoas, nun README.md

## 1. Explica métodos para 'abrir' unha consola/shell a un contenedor en execución.
Podese facer dende visual studio code coa opción "Attach Shell" ou usando "docker exec -it nomecontenedor /ruta_intérprete".
```shell
docker run ubuntu
docker ps
docker exec -it gallant_brown /bin/bash

```
## 2. No contenedor anterior (en execución), qué opciones tes que ter usado ó arrincalo para poder interactuar coas súas entradas e salidas
As opcións son "-it", xa que arrancan una consola coa cal podese interactuar co contenedor directamente.
```shell
docker run -it ubuntu
```
## 3. Cómo sería un ficheiro docker-compose para que dous contenedores se comuniquen entre si nunha rede só deles?
Primeiro habría uq crear a rede docker usando "docker network create nomerede".
Despois no ficheiro creo 2 contenedores coa imaxe de ubuntu e asignolles a rede creada.
Comando para crear a rede:
```shell
docker network create \
  --driver=bridge \
  --subnet=172.23.0.0/16 \
  --ip-range=172.23.0.0/24 \
  --gateway=172.23.0.254 \
  redeexamen

```
Archivo docker-compose.yml:
```shell
services:
  equipo1:
    container_name: equipo1
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    networks:
      - redeexamen
  equipo2:
    container_name: equipo2
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    networks:
      - redeexamen
networks:
  redeexamen:
    external: true
```
Para comprobar que hay conectividade entre as redes uso "docker inspect nomecontenedor" para mirar a súa ip, e despois dende o outro contenedor probo a fecer ping.
docker inspect.
```shell
docker inspect equipo1
```
IP equipo1:
```shell
...
"Gateway": "172.23.0.1",
"IPAddress": "172.23.0.3",
"IPPrefixLen": 16,
...
```
Proba conectividade (como non hai acceso a internet esta parte non podese facer porque apt update necesita internet):
```shell
docker exec -it equipo2 /bin/bash
apt update
apt install iputils-ping
ping 172.23.0.3
```
## 4. Qué tes que engadir ó ficheiro anterior para que un contenedor teña unha IP fixa?
Engadindo "ipv4addres: direcciónIP" no .yml en cada contenedor na sección de network.
```shell
services:
  equipo1:
    container_name: equipo1
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    networks:
      redeexamen:
        ipv4_address: 172.23.0.3

  equipo2:
    container_name: equipo2
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    networks:
      redeexamen:
        ipv4_address: 172.23.0.4
networks:
  redeexamen:
    external: true


```
## 5. Qué comando de consola podo usar para sabe-las ips dos contenedores anteriores?
Usando "docker inspect nomecontenedor" na mostra toda a información dun contenedor incluida a rede.
Commando:
```shell
docker inspect equipo1

```
Resultado (só a parte da rede):
```shell
"Networks": {
                "redeexamen": {
                    "IPAMConfig": {
                        "IPv4Address": "172.23.0.3"
                    },
                    "Links": null,
                    "Aliases": [
                        "equipo1",
                        "equipo1"
                    ],
                    "MacAddress": "02:42:ac:17:00:03",
                    "DriverOpts": null,
                    "NetworkID": "2dfc8a04e4bc09816420481b0864446743f8fd9e5b8b235289eae774bc4f7a63",
                    "EndpointID": "e31e1b3b87b8e359b3eb7af27169a1d0531615303db4746464e615c677a2eaa4",
                    "Gateway": "172.23.0.254",
                    "IPAddress": "172.23.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": [
                        "equipo1",
                        "f86fd0d98166"
                    ]
                }
```
## 6. Cál é a funcionalidade do apartado "ports" en docker compose?
E para indicarlle ao contenedor que portos vai a usar para cominucarse.
Exemplo (no docker-compose.yml):
```shell
ports:
      - 120:120
```
## 7. Para qué serve o rexistro CNAME? Pon un exemplo
A súa función e facer que un dominio sexa o alias de outro, básicamente serve para facer subdominos que estás asociados aos dominios do rexistro A.
Exemplo: subdominio.dominio.com
## 8. Cómo podo facer para que a configuración dun contenedor DNS non se borre se creo outro contenedor?
Agrego no docker-compose.yml una sección de dns.
Exemplo:
```shell
dns:
      - 172.23.0.3
```
## 9. Engade unha zoa tendaelectronica.int no teu docker DNS que teña:
- www á IP 172.16.0.1
- owncloud sexa un CNAME de www
- un rexistro de texto có contido "1234ASDF"
Comproba que todo funciona có comando "dig"
Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"
(o apartado 9 realízase na máquina virtual)

Primeiro creo dous carpetas chamadas zonas e conf.
```shell
mkdir conf zonas
```
### Carpeta conf
Na carpeta conf engado os archivos:
-named.conf
-named.conf.default-zones
-named.conf.local
-named.conf.options

```shell
touch named.conf named.conf.local named.conf.options named.conf.default-zones
```
#### Archivo named.conf
No archivo named.conf apunto aos archivos named.conf.local e named.conf.options
Archivo named.conf:
```shell
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```
#### Archivo named.conf.options
Neste archivo gardo a configuración dos forwarders e desactivo DNSSEC validation.
Archivo named.conf.options:
```shell
options {
	directory "/var/cache/bind";
	// Disable DNSSEC validation
        dnssec-validation no;
	forwarders {
		8.8.8.8;
		8.8.4.4;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};
```
#### Archivo named.conf.local
Neste archivo declaro as zonas e a base de datos onde gardo a súa configuación.
Archivo named.conf.local:
```shell
zone "tendaelectronica.int" {
	type master;
	file "/var/lib/bind/db.tendaelectronica.int";
	allow-query {
		any;
		};
	};

```
#### Archivo named.conf.default-zones
Neste archivo decláranse as zoas por defecto (incluso as de resolución inversa).
Archivo named.conf.default-zones:
```shell
// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};

```
### Carpeta zonas
Nesta carpeta só teño o archivo da base de datos da zoa tendaelectronica.int.

#### Archivo db.tendaelectronica.int
```shell
$TTL    604800
@       IN      SOA     tendaelectronica.int. root.tendaelectronica.int. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      tendaelectronica.int.
@       IN      A       172.23.0.3      
@       IN      AAAA    ::1             
@       IN      TXT     "1234ASDF"

tendaelectronica.int. IN  A	172.23.0.3
ns	IN	A	172.23.0.3
www	IN	CNAME	tendaelectronica.int.

```

### docker-compose.yml
Modifico o archivo "docker-compose.yml" cambiando o equipo1 por un servidor con bind9 e engadindo as carpetas zonas e conf como volúmenes, mentres no equipo2 engado a sección dns para que resolva o servidor con bind9.
Archivo docker-compose.yml:
```shell
services:
  equiposervidor1:
    container_name: equiposervidor1
    image: ubuntu/bind9
    platform: linux/amd64
    ports:
      - 120:120
    networks:
      redeexamen:
        ipv4_address: 172.23.0.3
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind

  equipo2:
    container_name: equipo2
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 172.23.0.3
    networks:
      redeexamen:
        ipv4_address: 172.23.0.4
networks:
  redeexamen:

```

### Probas
#### Primeiro fago unas probas simples con ping:
```shell
docker exec -it equipo2 /bin/bash
apt update
apt install iputils-ping
ping google.es
ping tendaelectronica.int
```
Proba ping con tendaelectronica.int:
```shell
ping tendaelectronica.int
PING tendaelectronica.int (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.061 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.063 ms
^C
--- tendaelectronica.int ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1064ms
rtt min/avg/max/mdev = 0.061/0.062/0.063/0.001 ms

```

Despois instalo o paquete dnsutils que conten a ferramenta dig:
```shell
apt install dnsultils
```


#### Probas dig:
```shell
dig tendaelectronica.int 

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> tendaelectronica.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22814
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 644a1663b3276d5d0100000067379c87121309488351168a (good)
;; QUESTION SECTION:
;tendaelectronica.int.		IN	A

;; ANSWER SECTION:
tendaelectronica.int.	604800	IN	A	172.23.0.3

;; Query time: 1 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Nov 15 19:09:59 UTC 2024
;; MSG SIZE  rcvd: 93
```
Agora pregunto por recursos concretos.
SOA:
```shell
dig tendaelectronica.int SOA

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> tendaelectronica.int SOA
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26118
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7f339b493b93b7660100000067379d622baa2a92b78173ae (good)
;; QUESTION SECTION:
;tendaelectronica.int.		IN	SOA

;; ANSWER SECTION:
tendaelectronica.int.	604800	IN	SOA	tendaelectronica.int. root.tendaelectronica.int. 2 604800 86400 2419200 604800

;; Query time: 1 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Nov 15 19:13:38 UTC 2024
;; MSG SIZE  rcvd: 118

```
TXT:
```shell
dig tendaelectronica.int TXT            

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> tendaelectronica.int TXT
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24217
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a13fc9ef463716c30100000067379da588b221567275487c (good)
;; QUESTION SECTION:
;tendaelectronica.int.		IN	TXT

;; ANSWER SECTION:
tendaelectronica.int.	604800	IN	TXT	"1234ASDF"

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Nov 15 19:14:45 UTC 2024
;; MSG SIZE  rcvd: 98
```

## logs compose servidor bind9
```shell
user@ubuntuanxo:~/examenav1_anxovazquez/compose$ docker compose up
[+] Running 2/0
 ✔ Container equipo2          Created                                      0.0s 
 ✔ Container equiposervidor1  Created                                      0.0s 
Attaching to equipo2, equiposervidor1
equiposervidor1  | Starting named...
equiposervidor1  | exec /usr/sbin/named -u "bind" "-g" ""
equiposervidor1  | 15-Nov-2024 19:18:26.623 starting BIND 9.18.28-0ubuntu0.24.04.1-Ubuntu (Extended Support Version) <id:>
equiposervidor1  | 15-Nov-2024 19:18:26.623 running on Linux x86_64 6.8.0-48-generic #48~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Oct  7 11:24:13 UTC 2
equiposervidor1  | 15-Nov-2024 19:18:26.623 built with  '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--disable-dependency-tracking' '--libdir=/usr/lib/x86_64-linux-gnu' '--sysconfdir=/etc/bind' '--with-python=python3' '--localstatedir=/' '--enable-threads' '--enable-largefile' '--with-libtool' '--enable-shared' '--disable-static' '--with-gost=no' '--with-openssl=/usr' '--with-gssapi=yes' '--with-libidn2' '--with-json-c' '--with-lmdb=/usr' '--with-gnu-ld' '--with-maxminddb' '--with-atf=no' '--enable-ipv6' '--enable-rrl' '--enable-filter-aaaa' '--disable-native-pkcs11' 'build_alias=x86_64-linux-gnu' 'CFLAGS=-g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/build/bind9-CTg8aa/bind9-9.18.28=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/bind9-CTg8aa/bind9-9.18.28=/usr/src/bind9-1:9.18.28-0ubuntu0.24.04.1 -fno-strict-aliasing -fno-delete-null-pointer-checks -DNO_VERSION_DATE -DDIG_SIGCHASE' 'LDFLAGS=-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now' 'CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=3'
equiposervidor1  | 15-Nov-2024 19:18:26.623 running as: named -u bind -g
equiposervidor1  | 15-Nov-2024 19:18:26.623 compiled by GCC 13.2.0
equiposervidor1  | 15-Nov-2024 19:18:26.623 compiled with OpenSSL version: OpenSSL 3.0.13 30 Jan 2024
equiposervidor1  | 15-Nov-2024 19:18:26.623 linked to OpenSSL version: OpenSSL 3.0.13 30 Jan 2024
equiposervidor1  | 15-Nov-2024 19:18:26.623 compiled with libuv version: 1.48.0
equiposervidor1  | 15-Nov-2024 19:18:26.623 linked to libuv version: 1.48.0
equiposervidor1  | 15-Nov-2024 19:18:26.623 compiled with libxml2 version: 2.9.14
equiposervidor1  | 15-Nov-2024 19:18:26.623 linked to libxml2 version: 20914
equiposervidor1  | 15-Nov-2024 19:18:26.623 compiled with json-c version: 0.17
equiposervidor1  | 15-Nov-2024 19:18:26.623 linked to json-c version: 0.17
equiposervidor1  | 15-Nov-2024 19:18:26.623 compiled with zlib version: 1.3
equiposervidor1  | 15-Nov-2024 19:18:26.623 linked to zlib version: 1.3
equiposervidor1  | 15-Nov-2024 19:18:26.623 ----------------------------------------------------
equiposervidor1  | 15-Nov-2024 19:18:26.623 BIND 9 is maintained by Internet Systems Consortium,
equiposervidor1  | 15-Nov-2024 19:18:26.623 Inc. (ISC), a non-profit 501(c)(3) public-benefit 
equiposervidor1  | 15-Nov-2024 19:18:26.623 corporation.  Support and training for BIND 9 are 
equiposervidor1  | 15-Nov-2024 19:18:26.623 available at https://www.isc.org/support
equiposervidor1  | 15-Nov-2024 19:18:26.623 ----------------------------------------------------
equiposervidor1  | 15-Nov-2024 19:18:26.623 found 2 CPUs, using 2 worker threads
equiposervidor1  | 15-Nov-2024 19:18:26.623 using 2 UDP listeners per interface
equiposervidor1  | 15-Nov-2024 19:18:26.623 DNSSEC algorithms: RSASHA1 NSEC3RSASHA1 RSASHA256 RSASHA512 ECDSAP256SHA256 ECDSAP384SHA384 ED25519 ED448
equiposervidor1  | 15-Nov-2024 19:18:26.623 DS algorithms: SHA-1 SHA-256 SHA-384
equiposervidor1  | 15-Nov-2024 19:18:26.623 HMAC algorithms: HMAC-MD5 HMAC-SHA1 HMAC-SHA224 HMAC-SHA256 HMAC-SHA384 HMAC-SHA512
equiposervidor1  | 15-Nov-2024 19:18:26.623 TKEY mode 2 support (Diffie-Hellman): yes
equiposervidor1  | 15-Nov-2024 19:18:26.623 TKEY mode 3 support (GSS-API): yes
equiposervidor1  | 15-Nov-2024 19:18:26.625 loading configuration from '/etc/bind/named.conf'
equiposervidor1  | 15-Nov-2024 19:18:26.625 unable to open '/etc/bind/bind.keys'; using built-in keys instead
equiposervidor1  | 15-Nov-2024 19:18:26.625 looking for GeoIP2 databases in '/usr/share/GeoIP'
equiposervidor1  | 15-Nov-2024 19:18:26.625 using default UDP/IPv4 port range: [32768, 60999]
equiposervidor1  | 15-Nov-2024 19:18:26.625 using default UDP/IPv6 port range: [32768, 60999]
equiposervidor1  | 15-Nov-2024 19:18:26.626 listening on IPv4 interface lo, 127.0.0.1#53
equiposervidor1  | 15-Nov-2024 19:18:26.626 listening on IPv4 interface eth0, 172.23.0.3#53
equiposervidor1  | 15-Nov-2024 19:18:26.626 IPv6 socket API is incomplete; explicitly binding to each IPv6 address separately
equiposervidor1  | 15-Nov-2024 19:18:26.626 listening on IPv6 interface lo, ::1#53
equiposervidor1  | 15-Nov-2024 19:18:26.626 generating session key for dynamic DNS
equiposervidor1  | 15-Nov-2024 19:18:26.627 sizing zone task pool based on 1 zones
equiposervidor1  | 15-Nov-2024 19:18:26.627 none:99: 'max-cache-size 90%' - setting to 3792MB (out of 4213MB)
equiposervidor1  | 15-Nov-2024 19:18:26.627 set up managed keys zone for view _default, file 'managed-keys.bind'
equiposervidor1  | 15-Nov-2024 19:18:26.628 configuring command channel from '/etc/bind/rndc.key'
equiposervidor1  | 15-Nov-2024 19:18:26.628 command channel listening on 127.0.0.1#953
equiposervidor1  | 15-Nov-2024 19:18:26.628 configuring command channel from '/etc/bind/rndc.key'
equiposervidor1  | 15-Nov-2024 19:18:26.628 command channel listening on ::1#953
equiposervidor1  | 15-Nov-2024 19:18:26.628 not using config file logging statement for logging due to -g option
equiposervidor1  | 15-Nov-2024 19:18:26.628 managed-keys-zone: loaded serial 0
equiposervidor1  | 15-Nov-2024 19:18:26.629 zone tendaelectronica.int/IN: loaded serial 2
equiposervidor1  | 15-Nov-2024 19:18:26.629 all zones loaded
equiposervidor1  | 15-Nov-2024 19:18:26.629 running
```







