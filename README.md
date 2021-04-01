# Comandos-Linux-SX
##Practica 1: Configuración inicial    
Fichero configuración interficies de red: 
`/etc/network/interfaces`    
Reiniciar servicios de red:
`systemctl restart networking`  
Fichero para activar forwarding:  
`/etc/sysctl.conf`  
Regla para NAT:  
`iptables -t nat -A POSTROUTING ! -d 192.168.2.0/24 -o eth1 -j MASQUERADE`   
Fichero servidor DNS:  
`/etc/resolv.conf`  
##Practica 2: Autoridad de Certificación  
Fichero de configuración de OpenSSL:  
`/etc/ssl/openssl.cnf`  
Comandos generales de OpenSSL:  



