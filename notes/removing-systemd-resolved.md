I was having a ton of isuses resolving external DNS on my Ubuntu nodes. I was consistently seeing connection refused to 127.0.0.53. After some digging around, I discovered it was related to systemd-resolved

systemctl disable systemd-resolved.service

service systemd-resolved stop

rm /etc/resolv.conf

cat << EOF > /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
EOF

service systemd-networkd restart