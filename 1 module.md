## ISP

```bash
hostnamectl set-hostname isp.au-team.irpo; exec bash
```

```bash
timedatectl set-timezone Asia/Yekaterinburg
```

```bash
mkdir -p /etc/net/ifaces/ens2 /etc/net/ifaces/ens3
```

```bash
cat > /etc/net/ifaces/ens2/options << EOF
BOOTPROTO=static
TYPE=eth
DISABLED=no
CONFIG_IPV4=yes
NM_CONTROLLED=no
EOF
```

```bash
echo "172.16.1.1/28" > /etc/net/ifaces/ens2/ipv4address
```

```bash
cat > /etc/net/ifaces/ens3/options << EOF
BOOTPROTO=static
TYPE=eth
DISABLED=no
CONFIG_IPV4=yes
NM_CONTROLLED=no
EOF
```

```bash
echo "172.16.2.1/28" > /etc/net/ifaces/ens3/ipv4address
```

```bash
vim /etc/net/sysctl.conf
```

```bash
sysctl -p 
```

```bash
systemctl restart network
```

```bash
apt-get update && apt-get install -y iptables tzdata
```

```bash
iptables -t nat -A POSTROUTING -o ens1 -s 172.16.1.0/28 -j MASQUERADE
```

```bash
iptables -t nat -A POSTROUTING -o ens1 -s 172.16.2.0/28 -j MASQUERADE
```

```bash
iptables -A FORWARD -i ens2 -o ens1 -s 172.16.1.0/28 -j ACCEPT
```

```bash
iptables -A FORWARD -i ens3 -o ens1 -s 172.16.2.0/28 -j ACCEPT
```

```bash
iptables -A FORWARD -s 172.16.0.0/16 -d 172.16.0.0/16 -j ACCEPT
```

```bash
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
```

```bash
iptables-save > /etc/sysconfig/iptables
```

```bash
systemctl enable --now iptables
```

## HQ-RTR

```bash
hostnamectl set-hostname hq-rtr.au-team.irpo; exec bash
```

```bash
timedatectl set-timezone Asia/Yekaterinburg
```

```bash
mkdir -p /etc/net/ifaces/ens1 /etc/net/ifaces/ens2 /etc/net/ifaces/ens2.100 /etc/net/ifaces/ens2.200 /etc/net/ifaces/ens2.999
```

```bash
cat > /etc/net/ifaces/ens1/options << EOF
BOOTPROTO=static
TYPE=eth
DISABLED=no
CONFIG_IPV4=yes
NM_CONTROLLED=no
EOF
```

```bash
echo "172.16.1.2/28" > /etc/net/ifaces/ens1/ipv4address
```

```bash
echo "default via 172.16.1.1" > /etc/net/ifaces/ens1/ipv4route
```

```bash
echo "nameserver 77.88.8.7" > /etc/net/ifaces/ens1/resolv.conf
```

```bash
cat > /etc/net/ifaces/ens2/options << EOF
BOOTPROTO=none
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
EOF
```

```bash
cat > /etc/net/ifaces/ens2.100/options << EOF
BOOTPROTO=static
TYPE=vlan
VID=100
HOST=ens2
DISABLED=no
CONFIG_IPV4=yes
NM_CONTROLLED=no
EOF
```

```bash
echo "192.168.100.1/27" > /etc/net/ifaces/ens2.100/ipv4address
```

```bash
cat > /etc/net/ifaces/ens2.200/options << EOF
BOOTPROTO=static
TYPE=vlan
VID=200
HOST=ens2
DISABLED=no
CONFIG_IPV4=yes
NM_CONTROLLED=no
EOF
```

```bash
echo "192.168.200.65/28" > /etc/net/ifaces/ens2.200/ipv4address
```

```bash
cat > /etc/net/ifaces/ens2.999/options << EOF
BOOTPROTO=static
TYPE=vlan
VID=999
HOST=ens2
DISABLED=no
CONFIG_IPV4=yes
NM_CONTROLLED=no
EOF
```

```bash
echo "192.168.99.89/29" > /etc/net/ifaces/ens2.999/ipv4address
```

```bash
mkdir -p /etc/net/ifaces/gre1
```

```bash
cat > /etc/net/ifaces/gre1/options << EOF
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.1.2
TUNREMOTE=172.16.2.2
TUNOPTIONS='ttl 64'
DISABLED=no
CONFIG_IPV4=yes
EOF
```

```bash
echo "10.10.10.1/30" > /etc/net/ifaces/gre1/ipv4address
```

```bash
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/' /etc/net/sysctl.conf
```

```bash
sysctl -p 
```

```bash
systemctl restart network
```

```bash
useradd -m -s /bin/bash net_admin
```

```bash
echo "net_admin:P@ssw0rd" | chpasswd
```

```bash
usermod -a -G wheel net_admin
```

```bash
echo "net_admin ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/net_admin
```

```bash
chmod 440 /etc/sudoers.d/net_admin
```

```bash
apt-get update && apt-get install -y iptables tzdata sudo frr dhcp-server nano
```

```bash
iptables -t nat -A POSTROUTING -o ens1 -s 192.168.100.0/27 -j MASQUERADE
```

```bash
iptables -t nat -A POSTROUTING -o ens1 -s 192.168.200.64/28 -j MASQUERADE
```

```bash
iptables -t nat -A POSTROUTING -o ens1 -s 192.168.99.88/29 -j MASQUERADE
```

```bash
iptables -A FORWARD -i ens2.100 -o ens1 -s 192.168.100.0/27 -j ACCEPT
```

```bash
iptables -A FORWARD -i ens2.200 -o ens1 -s 192.168.200.64/28 -j ACCEPT
```

```bash
iptables -A FORWARD -i ens2.999 -o ens1 -s 192.168.99.88/29 -j ACCEPT
```

```bash
iptables -A FORWARD -s 192.168.0.0/16 -d 192.168.0.0/16 -j ACCEPT
```

```bash
iptables -A FORWARD -s 10.10.10.0/30 -j ACCEPT
```

```bash
iptables -A FORWARD -d 10.10.10.0/30 -j ACCEPT
```

```bash
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
```

```bash
iptables-save > /etc/sysconfig/iptables
```

```bash
systemctl enable --now iptables
```

```bash
sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons
```

```bash
systemctl enable --now frr
```

```bash
vtysh -c "conf t" \
 -c "router ospf" \
 -c "ospf router-id 1.1.1.1" \
 -c "network 10.10.10.0/30 area 0" \
 -c "network 192.168.100.0/27 area 0" \
 -c "network 192.168.200.64/28 area 0" \
 -c "network 192.168.99.88/29 area 0" \
 -c "passive-interface default" \
 -c "no passive-interface gre1" \
 -c "exit" \
 -c "interface gre1" \
 -c "ip ospf authentication message-digest" \
 -c "ip ospf message-digest-key 1 md5 P@ssw0rd" \
 -c "exit" \
 -c "exit" \
 -c "wr"
```

```bash
cat > /etc/dhcp/dhcpd.conf << EOF
subnet 192.168.200.64 netmask 255.255.255.240 {
    option routers                  192.168.200.65;
    option subnet-mask              255.255.255.240;
    option domain-name              "au-team.irpo";
    option domain-name-servers      192.168.100.2;
    range dynamic-bootp 192.168.200.66 192.168.200.78;
    default-lease-time 600;
    max-lease-time 7200;
}
EOF
```

```bash
echo 'DHCPDARGS="ens2.200"' > /etc/sysconfig/dhcpd
```

```bash
systemctl enable --now dhcpd
```

```bash
sed -i 's/nameserver 77.88.8.7/nameserver 192.168.100.2/' /etc/net/ifaces/ens1/resolv.conf
```

```bash
systemctl restart network
```

## HQ-SRV

```bash
hostnamectl set-hostname hq-srv.au-team.irpo; exec bash
```

```bash
timedatectl set-timezone Asia/Yekaterinburg
```

```bash
mkdir -p /etc/net/ifaces/ens1
```

```bash
cat > /etc/net/ifaces/ens1/options << EOF
BOOTPROTO=static
TYPE=eth
DISABLED=no
CONFIG_IPV4=yes
NM_CONTROLLED=no
EOF
```

```bash
echo "192.168.100.2/27" > /etc/net/ifaces/ens1/ipv4address
```

```bash
echo "default via 192.168.100.1" > /etc/net/ifaces/ens1/ipv4route
```

```bash
echo "nameserver 77.88.8.7" > /etc/net/ifaces/ens1/resolv.conf
```

```bash
systemctl restart network
```

```bash
useradd sshuser -u 2027 -U -m -s /bin/bash
```

```bash
echo "sshuser:P@ssw0rd" | chpasswd
```

```bash
usermod -a -G wheel sshuser
```

```bash
echo "sshuser ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/sshuser
```

```bash
chmod 440 /etc/sudoers.d/sshuser
```

```bash
apt-get update && apt-get install -y openssh-server bind nano
```

```bash
cat > /etc/openssh/sshd_config << EOF
Port 2027
MaxAuthTries 2
Banner /etc/openssh/sshd_banner
AllowUsers sshuser
EOF
```

```bash
echo "Authorized access only" > /etc/openssh/sshd_banner
```

```bash
systemctl enable --now sshd
```

```bash
systemctl restart sshd
```

```bash
cat > /etc/bind/options.conf << EOF
options {
    directory "/etc/bind/zone";
    listen-on { 127.0.0.1; 192.168.100.2; };
    allow-query { any; };
    allow-recursion { any; };
    forward first;
    forwarders { 77.88.8.7; 77.88.8.3; };
};
EOF
```

```bash
cat > /etc/bind/local.conf << EOF
zone "au-team.irpo" {
    type master;
    file "/etc/bind/zone/db.au-team.irpo";
};

zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zone/db.192.168.100";
};

zone "200.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zone/db.192.168.200";
};

zone "3.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zone/db.192.168.3";
};
EOF
```

```bash
cat > /etc/bind/zone/db.au-team.irpo << EOF
\$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. root.au-team.irpo. (
        2026112101 ; serial
        3600       ; refresh
        1800       ; retry
        604800     ; expire
        86400 )    ; minimum

    IN  NS  hq-srv.au-team.irpo.

hq-rtr   IN  A   192.168.100.1
br-rtr   IN  A   192.168.3.17
hq-srv   IN  A   192.168.100.2
hq-cli   IN  A   192.168.200.66
br-fw    IN  A   192.168.3.1
br-srv   IN  A   192.168.3.2
EOF
```

```bash
cat > /etc/bind/zone/db.192.168.100 << EOF
\$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. root.au-team.irpo. (
        2026112101 3600 1800 604800 86400 )
    IN  NS  hq-srv.au-team.irpo.

1   IN  PTR  hq-rtr.au-team.irpo.
2   IN  PTR  hq-srv.au-team.irpo.
EOF
```

```bash
cat > /etc/bind/zone/db.192.168.200 << EOF
\$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. root.au-team.irpo. (
        2026112101 3600 1800 604800 86400 )
    IN  NS  hq-srv.au-team.irpo.

66  IN  PTR  hq-cli.au-team.irpo.
EOF
```

```bash
cat > /etc/bind/zone/db.192.168.3 << EOF
\$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. root.au-team.irpo. (
        2026112101 3600 1800 604800 86400 )
    IN  NS  hq-srv.au-team.irpo.

1   IN  PTR  br-fw.au-team.irpo.
2   IN  PTR  br-srv.au-team.irpo.
17  IN  PTR  br-rtr.au-team.irpo.
EOF
```

```bash
chown -R root:named /etc/bind/zone
```

```bash
named-checkconf -z
```

```bash
systemctl enable --now bind
```

```bash
systemctl restart bind
```

```bash
rm -f /etc/net/ifaces/ens1/resolv.conf
```

```bash
echo "name_servers=127.0.0.1" > /etc/resolvconf.conf
```

```bash
resolvconf -u
```

```bash
systemctl restart network
```

## HQ-CLI
