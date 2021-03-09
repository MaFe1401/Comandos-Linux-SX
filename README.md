# Comandos-Linux-SX
/etc/network/interfaces                                                  | Fichero configuración interficies de red
systemctl restart networking                                             | Reiniciar servicios de red
/etc/sysctl.conf                                                         | Fichero activar forwarding
iptables -t nat -A POSTROUTING ! -d 192.168.2.0/24 -o eth1 -j MASQUERADE | Regla para NAT
/etc/resolv.conf                                                         | Fichero DNS
