# Comandos-Linux-SX
## Practica 1: Configuración inicial    
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
## Practica 2: Autoridad de Certificación  
Fichero de configuración de OpenSSL:  
`/etc/ssl/openssl.cnf`  
Comandos generales de OpenSSL:  
* Para gestionar una AC:  
`openssl ca`  
* Para gestionar la creación de peticiones de certificado u otras:  
`openssl req`  
Generar petición de certificado:  
`openssl req -config openssl.cnf -new -keyout private/cakey.pem -out requests/careq.pem`  
La petición se genera usando el fichero de configuración `openssl.cnf`, la clave privada se guarda en `private/cakey.pem` y la petición de certificado se guarda en `requests/careq.pem`.  
Firmar el certificado (tendremos un certificado autofirmado. Lo ha generado la CA (nosotros) y lo ha firmado la CA (nosotros)):  
`openssl ca -config openssl.cnf -extensions v3_ca -days 3652 -create_serial -selfsign -in requests/careq.pem -out cacert.pem`  

