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
La firma se realiza usando el fichero de configuración `openssl.cnf`, el certificado tiene una validez de 3652 dias, se crea un numero de serie con `-create_serial`, se autofirma con `-selfsign`, la petición que se firma se encuentra en `requests/careq.pem` y el certificado firmado se llama `cacert.pem`.  
### Información adicional  
El comando `openssl ca` necesita una estructura de ficheros como la siguiente para funcionar:  
* `certs/` - Directorio donde se guardan los certificados.  
* `crl/` - Directorio donde se guardan las CRLS generadas.  
* `newcerts/` - Directorio donde OpenSSL guarda los certificados generados como ficheros pem con el nombre <#serial>.pem., es decir, el nombre del fichero coincide con el número de serie del certificado. Como nuestra AC es una AC raíz (con un certificado auto-firmado), el primer certificado que va a aparecer en esta sección será el de la AC.  
* `private/` - Directorio que almacena la clave privada de la AC.  
* `private/cakey.pem` – Fichero que contiene la clave privada de la AC  
* `cacert.pem` - Certificado de la AC.  
* `crlnumber` – Número actual de la CRL.  
* `index.txt` – Base de datos (en formato texto) que contiene la información de los certificados emitidos y revocados.  
* `serial` – Almacena el siguiente número de serie en formato hexadecimal.  

Acciones importantes a realizar en el fichero `openssl.cnf`:  
* En el apartado `[CA_default]`, cambiar el directorio de la AC a `.` , así OpenSSL buscará los ficheros y directorios de la AC en el directorio donde se ejecuta: `dir = .`  
* En `[CA_default]`, descomentar `copy_extensions = copy`, que permite añadir las extensiones de una petición de certificado al certificado final.  
* En `[req]`, cambiar la longitud de la clave RSA a 4096: `default_bits = 4096`.  
* Modificar la sección `[v3_ca]`, que define las extensiones para una AC típica:  
```
basicConstraints = critical,CA:true  
subjectKeyIdentifier=hash  
authorityKeyIdentifier=keyid:always,issuer  
keyUsage = critical, cRLSign, keyCertSign  

```

