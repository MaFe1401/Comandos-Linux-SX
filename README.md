# Comandos-Linux-SX
## Práctica 1: Configuración inicial    
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
## Práctica 2: Autoridad de Certificación  
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
## Práctica 3: Implementación de un servidor HTTPS  
En el fichero `openssl.cnf`, crear una nueva sección `server_cert`:  
```  
basicConstraints = CA:FALSE  
subjectKeyIdentifier = hash  
authorityKeyIdentifier = keyid, issuer  
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment, keyAgreement  
extendedKeyUsage = critical, serverAuth  
```  
Crear la clave privada y la petición (es solo de prueba, para luego comprobar la base de datos `index.txt`, revocarlo y comprobar la Certificate Revocation List):    
`openssl req -new -keyout private/testserver.key.pem -out requests/testserver.csr.pem`  
Firmar el certificado con el comando:  
`openssl ca -config openssl.cnf -extensions server_cert -in requests/testserver.csr.pem -out certs/testserver.crt.pem`  
Comprobar que el certificado es correcto:  
`openssl verify -CAfile cacert.pem certs/testserver.crt.pem`  
Esto verifica que la firma es correcta usando el certificado de la AC `cacert.pem`.  
Revocar el certificado:  
`openssl ca -config openssl.cnf -revoke certs/testserver.crt.pem`  
Generar la CRL:  
`openssl ca -gencrl -config openssl.cnf -out ./crl/crl.pem`  
Usando la configuración del fichero `openssl.cnf`, se genera la lista de certificados revocados en `/home/kali/rootca/crl/crl.pem`.  
Observar el contenido de la CRL:  
`openssl crl -noout -text -in ./crl/crl.pem`  
Comprobar la validez del certificado (deberia decir que no lo es ya que lo hemos revocado):  
`openssl verify -CAfile cacert.pem -crl_check -CRLfile ./crl/crl.pem certs/testserver.crt.pem`  
Generar el par de claves RSA para el servidor y la petición (ahora ya sí):  
`openssl req -new -addext 'subjectAltName = IP:10.0.2.10' -nodes -keyout tls/webserver.key.pem -out webserver.csr.pem`  
La extensión `subjectAltName` permite especificar nombres de host adicionales que también serán protegidos por el certificado, en este caso se especifica la IP del servidor, que es la de nuestra máquina virtual. El par de claves se guarda en `tls/webserver.key.pem` que se encuentra dentro de la carpeta del servidor que en nuestro caso es `/webserver`. La ruta completa sería `webserver/tls/webserver.key.pem`. La petición de certificado se llama `webserver.csr.pem`. La opción `nodes` elimina el cifrado del fichero que contiene la clave privada del servidor (mediante un password), así cuando el servidor necesite acceder a su clave privada no habrá que introducir manualmente dicho password.  
Firmar la petición que hemos generado (para ello antes hay que haber guardado la petición en la carpeta `requests` de `rootca` que es donde se encuentran todos los ficheros de nuestra CA):  
`openssl ca -config openssl.cnf -extensions server_cert -in requests/webserver.csr.pem -out certs/webserver.crt.pem`  
Mover el certificado que se acaba de guardar en `certs/webserver.crt.pem` a la carpeta del servidor `/webserver/tls`.  
Instalar node.js en la máquina virtual:  
`sudo apt update && sudo apt install npm`  
Crear el fichero `/webserver/index.js` para crear el servidor https. Contenido del fichero:  
```  
'use strict';  
const express = require('express');  
const logger = require('morgan');  
const https = require('https');
const fs = require('fs');
const tlsServerKey = fs.readFileSync('./tls/webserver.key.pem'); //Clave  
const tlsServerCert = fs.readFileSync('./tls/webserver.crt.pem'); //Certificado  
const app = express();  
app.use(logger('dev'));  
app.get('/', (request, response) => {  
response.send('<h1>soy mongolito</h1>);  
});  
const httpOptions = {  
key: tlsServerKey,  
cert: tlsServerCrt  
};  
var server = https.createServer(httpsOptions, app);  
server.listen(443);  
server.on('listening', onListening);  
function onListening() {  
const addr = server.address();  
const bind = typeof addr === 'string'  
? 'pipe ' + addr  
: 'port ' + addr.port;  
console.log('Listening on ' + bind);  
}  
```  
Inicializar el proyecto:  
`npm init`  
Descargar e instalar express y morgan:  
`npm install express morgan`  
Arrancar el servidor:  
`sudo node index.js`  


