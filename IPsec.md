# Práctica de IPsec | parte 1 [ tunnel ]

Práctica de IPsec, parte 1.

# Previo:

Se cambia contenido del fichero `etc/ipsec.conf`:

```
file: /etc/ipsec.conf

config setup
	charondebug="all"

conn myvpn
	type=tunnel		# tunnel mode
	auto=start
	keyexchange=ikev1	# usando la version 1 del protocolo IKE
	##################
	#subnets and nets#
	##################
	ike=aes256-sha1-modp3072
	esp=aes256-sha1
	aggressive=no		# main mode
```

Usando `man ipsec`, podremos obetener más info:

```
explicación de campos
```
Ahora en `/etc/ipsec.secrets` se almacenan los secretos compartidos con los demás peers:
Se añade la siguiente linea al fichero: 

```
peer_IP address : PSK "shared_secret"	# donde peer_IP address es la IP del router remoto y el
					# shared_secret es aquel valor del secreto compartido
```
ahora se puede general el tunnel IPsec con el comando `ipsec start --nofork`

## Ejercicio 1 - Análisis del protocolo IKEv1

`Cabe destacar en este caso que el dialogo IKE se va a establecer 2 veces consecutivamente y se peuden ver ya que el último mensaje de la cadena es un informational, el cual se usa para borrar las diferentes SAs configuradas`

### Modo de túnel

En este caso se intercambia el dialogo en modo main, y se puede observar un flujo de 6 mensajes IKEv1 entrecortados por 1 mensaje Informational

### Análisis del contenido

`Se obvservan los campos IC, RC, nonce [de cada peer] y finalmente el intercambio diffie-helmann`

(nos fijaremos solo en el primer flujo que hay)
<strong> FLUJO 1 </strong>

Este intercambio ha sido iniciado por R2 [10.0.2.20] y el contenido de los campos son:
* IC=037db2d2145dfc70
* RC=535401f4f0ac24db
* nonce_r2=7b307e4446dd7a92c41cbddc6a14f632c5f7ce3bc73953f9dea4ddeb3428bed6
* nonce_r1=9975d8f244c72158b46ca9c46b13690653fa10ffd9554fb9df8c806f20f8b2de
* Intercambio Diffie-Helmann:
	- R2: key_exchange_value=1e3591752b2848b2231b339f275c8581c973a58f816d61be60514100ce63dadd88c69d512bfb2e2d9622e97434c2f27d8663ff9ba25d6160e8430b4d38814ebb9e51249e086f798e7f724b6778f695ac5920f85f08b57c8eb8acc842b6c5b682294838c3f263cc6bf05d56bf9519d4420c641adb04c6ea062579d02df1bf355ab455c7b397191fa6ed60e01acca9a40a959d686cb031f0d3eb60fd58cfc12130c6e52957ef0756c6b6a7046ff0ef30be4b8528ac633e62732278d7c6bc0eff94a1830382b1692a99b7dda4c727d4171c36f6b62cd0b80ef4922fe90cd307b58d61a9b4c7f7383ac6b6914354c6ba26e980da3c8473b7df3ec4daba919f544a9ffacc9df198936d96f7d3d03ddb671f718d1faa93ae89f45b2d16129a03200487631b33a8a0bc77cadeacb9525265b754e2ccbf4d5c7463c491c941a32d68219dd9b2b0032417dc0d40d954515114f2ea51d9d76ae40bbcf1db0fa61eb440c92d43c5b8259294d51b329d78e6afb6881b5dc5dc6f8b877cf2da48e512e9740c16
	- R1: key_exchange_value=c34ae9b98b7db05ef9fd99bb18d7aab1b2cf620da4a4e2e3f4d7f1957591c2e698a7183c5b6dd457f75b8850234ede87da81cd65fc4bcb4426c8cd06bf87a34e50382f50d49d0da95c9af34f1cf5fc10687900478e0d801e3c39f70bd89ef07fc3b95df8bccad624a1758a2b0aea012c3bfe557a05de28c2bca8176a744f1721114c6c68c5c5013be4645ad89ba9fd56aa4c7132165305fe2e9c80c36d482dca74bf4de0aad60527c5f3861096ab905b71a83847385ebb57bf8914637b422063196c4385c09c2f10d49f471783737d3d3beb1228033c1b334eee5442a1e2b88a821504e02288cd89f96f886fb54c88a18ba2982a53708b1623908f93cac84a3a4c13599402db6413f10d6b344fc25c1419a8a9a036aca2da524962648cc732c9ff476f8a7662c197cb98792b41169acd6d1f177eac9eafbf24b97d52c9ea3187ee399f1bcd084540e114798ab20a60dc7cb71074a75455f7c0fb11a8583da6040c33176d37c4a375c88fdb9d091c02faf569b19b13d579a1a214497101c4016d

### Identidad de los peers:

Las identidades de cada peer se transmiten una vez se ha hecho el key exchange con Diffie-Helmann con tal de poder encryptar esa misma identidad. Con lo cual no se puede ver.

### Fase de 2 del protocolo IKE:

Solo hay 3 mensajes de fase 2 conocidos como Quick Mode. En este paquete, no, no se puede ver el contenido del mismo ya que está encriptado con los parametros negociados en las IKE SAs

Justamente esta fase sirve para determinar las SAs que se van a usar para cada uno de los peers que usen el túnel.

## Ejercicio 2 - ESP
