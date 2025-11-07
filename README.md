# zabbix-haproxy
Come settare zabbix con haproxy mariadb cluster frontend e backend separati

Settare il cluster di mariadb con haproxy usando source come bilanciatore (ip da usare sia per il backend che frontend)

es:

frontend mariadb-front-sticky
  mode tcp <br />
  option  tcplog
  bind 192.168.xxx.214:3306
  default_backend mariadb-back-sticky
  


backend mariadb-back-sticky
  option  mysql-check user haproxy post-41
  mode tcp
  balance source
  server mariadb-1 192.168.xxx.21:3306 check
  server mariadb-2 192.168.xxx.22:3306 check
  server mariadb-3 192.168.xxx.23:3306 check
  server mariadb-4 192.168.xxx.24:3306 check


#settare haproxy per il controllo del server di backup attivo sul frontend
#es:
frontend zabbix-server-front
  mode tcp
  option  tcplog
  bind 192.168.xxx.215:10051
  default_backend zabbix-server-back


backend zabbix-server-back
  mode tcp
  balance source
  server zabbixb-1 192.168.xxx.26:10051 check
  server zabbixb-2 192.168.xxx.27:10051 check


########### zabbix web
frontend zabbix-front
   monitor-uri /checkstatus
   monitor fail if { nbsrv(zabbix-back) eq 0 }
   bind 192.168.xxx.213:80 # -> ip da richiamare nel browser
   mode http
   default_backend zabbix-back
   option httplog


backend zabbix-back
   option httpchk GET /zabbix-web #-> file messo nella root del sito
   balance roundrobin
   mode http
   cookie SERVER insert indirect nocache
   server zabbix1 192.168.xxx.29:80 check cookie zabbix1
   server zabbix2 192.168.xxx.28:80 check cookie zabbix2




#Creare i due o più frontend
#pacchetti da installare
apt install zabbix-nginx-conf
apt install zabbix-web-service
apt install zabbix-frontend-php
#comandi
cp /etc/zabbix/nginx.conf /etc/nginx/sites-availabe/zabbix.conf
##modificare zabbix.conf
#listen 80;
#server_name _;
cd /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-availabe/zabbix.conf zabbix.conf
#usare il configuratore di zabbix, usare il vip usato in haproxy per il cluster mariadb (192.168.xxx.214)
#es (zabbix.conf.php):
$DB['TYPE']                     = 'MYSQL';
$DB['SERVER']                   = '192.168.xxx.214';


#nel file zabbix.conf.php usare questi valori
$ZBX_SERVER                     = '192.168.xxx.215'; #-> vip da haproxy per il controllo del backend di zabbix
$ZBX_SERVER_PORT                = '10051';

#nel zabbix_server.conf indirizzo ip del db
DBHost=192.168.xxx.214


