# bind
bind zone transfer and views

/etc/bind/named.conf.local (NS1)
acl update { key updatekey; };
acl internal { !key external; key internal; 172.16.0.0/12; 10.0.0.0/8; 192.168.0.0/16; localhost; };
acl external { !key internal; key external; 0.0.0.0/0; };

view "internal" {
    match-clients { !key updatekey; internal; };
        zone "yourzone.cz" {
             type master;
             file "/etc/bind/internal/db.yourzone.cz";
                allow-update { key internal; } ;
                allow-transfer { key internal; };
                also-notify { NS2.IP key internal; };
        };
};


view "external" {
        match-clients { key external; key updatekey; any; };
        zone "yourzone.cz" {
             type master;
             file "/etc/bind/external/db.yourzone.cz";
                allow-transfer { key external; };
                allow-update { key external; } ;
                also-notify { NS2.IP key external; };
        };
};


/etc/bind/named.conf.options (NS1)
options {
        directory "/var/cache/bind";

        forwarders {
                8.8.8.8; 1.1.1.1;
         };

        dnssec-validation auto;
        allow-recursion { localhost; 172.16.0.0/12; 10.0.0.0/8; 192.168.0.0/16; };
        allow-query { any; };
        notify explicit;
        auth-nxdomain no;    # conform to RFC1035
        listen-on { NS1.IP; };
        listen-on-v6 { none; };
        server-id none;
        version "Secured DNS server";
};

statistics-channels {
  inet NS1.IP port 8053;
};


/etc/bind/named.conf.local (NS2)
acl internal { !key external; key internal; 172.16.0.0/12; 10.0.0.0/8; 192.168.0.0/16; localhost; };
acl external { !key internal; key external; 0.0.0.0/0; };

view "internal" {
    match-clients { internal; };
        zone "yourzone.cz" {
             type slave;
             file "/var/cache/bind/db.yourzone.cz.internal";
                masters { NS1.IP key internal; };
                allow-update { key "internal"; } ;
        };
        

view "external" {
        match-clients { external; };
        zone "yourzone.cz" {
             type slave;
             file "/var/cache/bind/db.yourzone.cz.external";
                masters { NS1.IP key external; };
                allow-update { key "external"; } ;
        };
       
};


/etc/bind/named.conf.options (NS2)
options {
        directory "/var/cache/bind";

         forwarders {
         8.8.8.8; 1.1.1.1;
         };

        dnssec-validation auto;
        allow-recursion { localhost; 172.16.0.0/12; 10.0.0.0/8; 192.168.0.0/16; };
        allow-query { any; };
        notify explicit;
        listen-on-v6 { none; };
        server-id none;
        version "Secured DNS server";
        auth-nxdomain no;    # conform to RFC1035
        listen-on { NS2.IP; };
};

statistics-channels {
  inet NS2.IP port 8053;
};
