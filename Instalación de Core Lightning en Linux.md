---
title: Instalación de Core Lightning en Linux
tags: [bitcoin, cli, nodo, node, macos]

---

# Instalación de Core Lightning (anteriormente C-Lightning) en Linux
###### tags: `bitcoin` `cli` `nodo` `node` `macos` `Linux` `Lightning`

Core Lightning (anteriormente C-lightning) es una implementación liviana, altamente personalizable y compatible con los estándares del protocolo de la red Lightning.
- Aquí tienes una guía paso a paso para instalar un nodo Lightning (usando Core Lightning como ejemplo) en Linux.


**Tabla de contenido**
[TOC]

## Autores
Jonny_Ji. 
Twitter para correcciones, comentarios o sugerencias: [@JonnyJi50127056](https://x.com/JonnyJi50127056)

El presente tutorial fue elaborado para el curso [Nodes from zero to hero](https://libreriadesatoshi.com/courses/nodes-from-zero-to-hero) a través de [@libreriadesatoshi](https://twitter.com/libdesatoshi).

En el siguiente enlace puedes encontrar la documentación de referencia:
[Core Lightning Documentation](https://docs.corelightning.org/docs/installation)

## Core Lightning
Core Lightning (CLN), anteriormente conocido como c-lightning, es una implementación ligera, modular y altamente personalizable del protocolo Lightning Network diseñada y mantenida por Blockstream.

Está escrita principalmente en C y utiliza un sistema de plugins que permite a desarrolladores y operadores adaptar y ampliar su funcionalidad según sus necesidades, destacándose por su cumplimiento estricto de los estándares BOLT para la interoperabilidad en la red.

**Características principales de Core Lightning**
- Ligereza y eficiencia: Uso mínimo de recursos, lo que permite operar múltiples instancias en hardware de bajo consumo como Raspberry Pi.
- Arquitectura modular: El uso de subdaemons y plugins facilita la personalización de funciones, integrando fácilmente nuevas capacidades o automatizando operaciones complejas.
- Privacidad: Soporta Multi-Part Payments por defecto, rutas aleatorias, y diversidad de canales, junto con soporte completo para Tor.
- Gestión avanzada de liquidez: Incluye herramientas para anuncios de liquidez P2P (Liquidity Ads) y soporte para aperturas colaborativas de canales dual-funding.
- Orientación al desarrollador: Mediante plugins en cualquier lenguaje y una API JSON-RPC muy potente, favorece integraciones avanzadas y experimentación.

**Enfoque de uso**
CLN es ideal para usuarios avanzados, desarrolladores y operadores que requieren un control granular sobre su nodo Lightning, personalización profunda, máxima eficiencia y reducida dependencia de soluciones monolíticas. Su enfoque abierto y flexible contribuye a la descentralización y evolución de Lightning Network, siendo compatible con otras implementaciones líderes como LND y Eclair.

## Instalar Programas y Dependencias
Instalar librerías necesarias para compilar:
```shell
sudo apt-get update
sudo apt-get install -y autoconf automake build-essential git libtool libgmp-dev libsqlite3-dev python3 python3-mako python3-pip net-tools zlib1g-dev libsodium-dev gettext jq
pip3 install --upgrade pip
```
Para usar el plugin `cln-grpc` o constuir plugins con `Rust` debe instalar Rust con esta linea:
```shell
sudo apt-get install -y cargo rustfmt
```
Para desarrollar, debes instalar estas dependencias adicionales:
```shell
sudo apt-get install -y valgrind libpq-dev shellcheck cppcheck libsecp256k1-dev jq
sudo apt-get install protobuf-compiler
```
- Si se muestra una advertencia de que un directorio no está nombrado en el PATH de Ubuntu, agregar la ruta del directorio al PATH, ejemplo:
```shell
export PATH=$PATH:/home/<tu_usuario>/.local/bin
```
## Instalar Core Lightning
Clonamos el repositorio donde está el código fuente de c-lightning y vamos a esa carpeta:
```shell
git clone https://github.com/ElementsProject/lightning.git
cd lightning
```
Buscamos la última versión estable del programa:
```shell
git tag
```
En el momento de actualizar este tutorial es la v23.05 de modo que vamos a ese release o al último lanzamiento que queramos instalar:
```shell
git checkout v23.08
```
Ahora si compilamos y ejecutamos lightning:
```shell
pip3 install --upgrade --no-warn-script-location pip # esta línea se agrega el no-warn para no ver advertencia
pip3 install mako 
./configure
make -j$(nproc)
sudo make install
```

Empieza la compilación y después de un tiempo ya está instalado Core Lightning. 
- Si falla la compilación, intenta nuevamente, pero agrega esta función en el comando `./configure` agregando esta orden: 
```shell
./configure --disable-rust
make -j$(nproc)
sudo make install
```
- Empieza nuevamente la compilación y después de un tiempo ya está instalado Core Lightning.

Antes de ejecutar `lightningd` debemos crear el directorio `.lightning` y dentro de ese directorio vamos a crear el archivo `config`.
```shell
sudo nano ~./lightning/config
```
Y agregamos las siguientes líneas:
```shell
network=bitcoin # Select the network parameters: bitcoin, testnet, testnet4, signet, or regtest
log-level=debug
log-file=lightningd.log
addr=0.0.0.0:9735
# database-upgrade=true  # está línea sólo en la versión 23.08, que necesita está orden
```
## Iniciar Lightningd
- Antes de lanzar `lightningd` ya debe estar corriendo y sincronizado `bitcoind`.

Lanzamos `lightningd` como demonio:
```shell
lightningd --daemon
```
Y obtenemos datos del nodo:
```shell
lightning-cli getinfo
```
Felicitaciones ahora tienes corriendo un nodo `core lightning`.

## Configurar lightningd para que use Tor
Para que nuestro nodo se conecte a través de Tor, solo debemos seguir estos pasos para agregar una dirección `.onion` a `lightningd`.
Abrimos el archivo de configuración de Tor:
```shell
sudo nano /etc/tor/torrc
```
y añadimos las siguientes líneas:
```shell
HiddenServiceDir /var/lib/tor/lightningd-service_v3/
HiddenServiceVersion 3
HiddenServicePort 9735 127.0.0.1:9735
```
presionamos `Ctrl+x` , luego presiona `‘s’` y enter para guardar los cambios.

Detenemos Tor:
```shell
sudo systemctl stop tor
```
Volvemos a lanzar Tor:
```shell
sudo systemctl start tor
```
Ahora ejecutamos el siguiente comando para obtener la dirección onion que nos asignó Tor.
```shell
sudo cat /var/lib/tor/lightningd-service_v3/hostname
```
>Responderá con una dirección onion para nuestro nodo LN, ejemplo:
bv4******************fxi7fs--*-*-*d.onion

- Esta dirección que nos acaba de arrojar es una dirección .onion fija para nuestro nodo lightning.

Añade esta dirección en el archivo de configuración de lightningd:
```shell
sudo nano ~./lightning/config
```
Copia las siguientes líneas en ese archivo reemplazando la que dice `announce-addr` por la que hayas obtenido:
```shell
announce-addr=hbv4******************fxi7fs--*-*-*d.onion:9735
proxy=127.0.0.1:9050
always-use-proxy=true
```
presiona `Ctrl+x` , luego presiona `‘s’` y enter para guardar los cambios.

Ahora detenemos lightningd con el siguiente comando:
```shell
lightning-cli stop
```
Ahora regarcamos Tor:
```shell
sudo systemctl restart tor
```
Y podemos iniciar de nuevo lightningd:
```shell
lightningd --daemon # es doble “-” antes de daemon
```
Para observar que esté funcionando lightningd, ejecutamos:
```shell
lightning-cli getinfo
```
Listo, tenemos un nodo Bitcoin y Core Lightning funcionando con Tor...

- Si en la actualización de los nodos, no borraste las carpetas `.bitcoin` y `.lightning`, los datos de los nodos permanecen intactos, incluidos la cadena de bloques y los datos de los canales lightning.


---

Si has llegado hasta aquí, felicitaciones, ahora tienes un nodo Bitcoin y Lightning listos para usar. Este es el primer pilar para comenzar a construir tu soberanía monetaria.

# Team "Librería de Satoshi"
