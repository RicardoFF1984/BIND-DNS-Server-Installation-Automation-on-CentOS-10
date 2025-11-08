#!/bin/bash
sudo dnf -y update
sudo dnf -y install bind bind-utils

interface2=$(ip -brief addr | grep UP | sed -n '2p' | awk '{print $1}')
  echo "Escolha a tua rede:"
  read rede

	echo "Escolha o teu ip:"
	read ip

	#Inserir Gateway
	echo "Defina o gateway:"
	read gateway

	#Inserir DNS
	echo "Escolha o teu DNS:"
	read dns

    sudo ip link set "$interface2" up
    sudo ip addr add "$ip" dev "$interface2"
    sudo ip route add default via "$gateway"

    echo "Qual é o teu domínio?"
    read domain

    echo " Que nome pretenedes dar à rede interna? (ex: internal-network) "
    read rede_interna

    octet1=$(echo "$ip" | cut -d'.' -f1)
    octet2=$(echo "$ip" | cut -d'.' -f2)
    octet3=$(echo "$ip" | cut -d'.' -f3)
    octet4=$(echo "$ip" | cut -d'.' -f4)

    ip2=$(echo "$ip" | cut -d'/' -f1)
	octet4_1=$(echo "$octet4" | cut -d'/' -f1)

    echo "
    acl $rede_interna {
    $rede;
}; 

options {
    listen-on port 53 { any; };             
    listen-on-v6 { any; };                   
    directory "\"/var/named\"";
    dump-file "\"/var/named/data/cache_dump.db\"";
    statistics-file "\"/var/named/data/named_stats.txt\"";
    memstatistics-file "\"/var/named/data/named_mem_stats.txt\"";
    secroots-file "\"/var/named/data/named.secroots\"";
    recursing-file "\"/var/named/data/named.recursing\"";

    allow-query { localhost; $rede_interna; };   
    allow-transfer { localhost; };                  

    recursion yes;

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };
    forward only;

    dnssec-validation yes;

    managed-keys-directory \""/var/named/dynamic\"";
    geoip-directory "\"/usr/share/GeoIP\"";

    pid-file "\"/run/named/named.pid\"";
    session-keyfile "\"/run/named/session.key\"";

    include "\"/etc/crypto-policies/back-ends/bind.config\"";
};

logging {
    channel default_debug {
        file "\"data/named.run\"";
        severity dynamic;
    };
};


zone "\"$domain\"" IN {
    type primary;
    file "\"$domain\"";
    allow-update { none; };
};

zone "\"$octet3.$octet2.$octet1.addr.arpa\"" IN {
    type primary;
    file "\"$octet3.$octet2.$octet1.db\"";
    allow-update { none; };
};

include "\"/etc/named.rfc1912.zones\"";
include "\"/etc/named.root.key\"";

" > named.txt

sudo mv named.txt /etc/named.conf

sudo echo -e "OPTIONS=\"-4\"" | sudo tee -a /etc/sysconfig/named


echo "
\$TTL 86400
@   IN  SOA     $domain. root.$domain. (
        2024122701  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
        IN  NS      $domain.
        IN  A       $ip2
        IN  MX 10   $domain.

kalaustudio   IN  A       $ip2
" > domain.txt
sudo mv domain.txt /var/named/"$domain"

echo "
\$TTL 86400
@   IN  SOA     $domain. root.$domain.com. (
        2024122701  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
        IN  NS      $domain.

$octet4_1     IN  PTR     $domain.
" > reverse.txt
sudo mv reverse.txt /var/named/"$octet3.$octet2.$octet1.db"

sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --remove-icmp-block=echo-reply --permanent


sudo chown root:named /etc/named.conf
sudo chmod 640 /etc/named.conf
sudo restorecon -v /etc/named.conf
sudo setenforce 0
sudo systemctl enable named.service
sudo systemctl start named.service
