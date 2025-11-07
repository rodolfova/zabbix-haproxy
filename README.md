# zabbix-haproxy
Come settare zabbix con haproxy mariadb cluster frontend e backend separati

Settare il cluster di mariadb con haproxy usando source come bilanciatore (ip da usare sia per il backend che frontend)

es:<br />
frontend mariadb-front-sticky<br />
  mode tcp <br />
  option  tcplog<br />
  bind 192.168.xxx.214:3306<br />
  default_backend mariadb-back-sticky<br />
  
backend mariadb-back-sticky<br />
  option  mysql-check user haproxy post-41<br />
  mode tcp<br />
  balance source<br />
  server mariadb-1 192.168.xxx.21:3306 check<br />
  server mariadb-2 192.168.xxx.22:3306 check<br />
  server mariadb-3 192.168.xxx.23:3306 check<br />
  server mariadb-4 192.168.xxx.24:3306 check<br />

###settare haproxy per il controllo del server di backup attivo sul frontend <br />
#es:<br />
frontend zabbix-server-front<br />
  mode tcp<br />
  option  tcplog<br />
  bind 192.168.xxx.215:10051<br />
  default_backend zabbix-server-back<br />

backend zabbix-server-back<br />
  mode tcp<br />
  balance source<br />
  server zabbixb-1 192.168.xxx.26:10051 check<br />
  server zabbixb-2 192.168.xxx.27:10051 check<br />

########### zabbix web<br />
frontend zabbix-front<br />
   monitor-uri /checkstatus<br />
   monitor fail if { nbsrv(zabbix-back) eq 0 }<br />
   bind 192.168.xxx.213:80 # -> ip da richiamare nel browser<br />
   mode http<br />
   default_backend zabbix-back<br />
   option httplog<br />


backend zabbix-back<br />
   option httpchk GET /zabbix-web #-> file messo nella root del sito<br />
   balance roundrobin<br />
   mode http<br />
   cookie SERVER insert indirect nocache<br />
   server zabbix1 192.168.xxx.29:80 check cookie zabbix1<br />
   server zabbix2 192.168.xxx.28:80 check cookie zabbix2<br />


#Creare i due o più frontend<br />
#pacchetti da installare<br />
apt install zabbix-nginx-conf<br />
apt install zabbix-web-service<br />
apt install zabbix-frontend-php<br />
#comandi<br />
cp /etc/zabbix/nginx.conf /etc/nginx/sites-availabe/zabbix.conf<br />
##modificare zabbix.conf<br />
listen 80;<br />
server_name _;<br />
cd /etc/nginx/sites-enabled<br />
ln -s /etc/nginx/sites-availabe/zabbix.conf zabbix.conf<br />
#usare il configuratore di zabbix, usare il vip usato in haproxy per il cluster mariadb (192.168.xxx.214)<br />
#es (zabbix.conf.php):<br />
$DB['TYPE']                     = 'MYSQL';<br />
$DB['SERVER']                   = '192.168.xxx.214';v


#nel file zabbix.conf.php usare questi valori<br />
$ZBX_SERVER                     = '192.168.xxx.215'; #-> vip da haproxy per il controllo del backend di zabbix<br />
$ZBX_SERVER_PORT                = '10051';<br />

#nel zabbix_server.conf indirizzo ip del db<br />
DBHost=192.168.xxx.214<br />


