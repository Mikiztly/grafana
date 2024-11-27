<h1 id="Servidor Prometheus y Grafana>">Servidor Prometheus y Grafana</h1>

Este repositorio lo configure con un servidor manejado con otro repositorio: [Mikiztly/portainer](https://github.com/Mikiztly/portainer) el cual tiene configurada una red interna llamada “lan-docker”, además utiliza [Nginx-Proxy-Manager](https://nginxproxymanager.com/) para manejar sub-dominios y así poder tener direcciones web en vez de IP's.<br>
Son varios archivos para poder montar un servidor con Prometheus y Grafana para monitorear los equipos que tengamos, además están los exporters:<br>

* [node_exporter](https://github.com/prometheus/node_exporter) para monitorizar sistemas Linux.
* [blackbox_exporter](https://github.com/prometheus/blackbox_exporter) para monitorizar páginas web.
* [cadvisor](https://github.com/google/cadvisor) para ver las estadísticas de docker.
* [ssl_exporter](https://github.com/ribbybibby/ssl_exporter) para monitorizar dentro de contenedores docker el certificado ssl.<br>

Para importar este repositorio en Portainer hay que poner la url del repositorio de la siguiente manera:<br>

![importar](/imagenes/importar.png)<br>

También hay que configurar las variables correctamente, es muy util GRAFANA_PLUGINS porque podemos agregar plugins como per ejemplo "alexanderzobnin-zabbix-app" para obtener datos desde Zabbix. Los nombres de los plugins se los separa con una coma (,).<br>

![Configurar Variables](/imagenes/variables.png)<br>

Para facilitar las variables aca deo una lista para copiar y pegar:
```shell
GRAFANA_URL=http://grafana.servidor.local/
GRAFANA_PLUGINS=grafana-clock-panel
GRAFANA_ADMIN=admin
GRAFANA_PASS=CambiaMe
DOCKER_PATH=/mnt/docker-data
```

Si se va a utilizar solo este repositorio hay que configurar bien la red y las variables.<br>

**IMPORTANTE**<br>
Como le agregue el mapeo de volúmenes a otro directorio para tener persistencia de datos, en el directorio configurado (variable DOCKER_PATH) tienen que existir las siguientes carpetas antes de levantar el stack:

```shell
prometheus/config
prometheus/data
grafana
```
Si no existen en el directorio (en mi caso /mnt/docker-data) va a dar error al levantar el stack.

<h1 id="Configuración>">Configuración</h1>

Hay cinco archivos de configuración que van en la carpeta **prometheus/config** que definimos en el docker:<br>

1) **prometheus.yml:** Este archivo tiene una configuración personalizada básica para obtener el monitoreo local con los 3 módulos, toma la lista de un archivo para obtener una lista de páginas web para monitorizar. Si alguna lista no existe no va a utilizar el exporter, por lo que es útil si queremos descartar algún monitor. **Este archivo no debe modificarse.**

2) **blackbox-targets.yml** contiene una lista de páginas que se van a monitorizar, se pueden agregar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Podemos monitorizar la URL, IP pública e IP privada.

3) **nodes-targets.yml** contiene una lista de servidores que se van a monitorizar, se pueden agregar con etiquetas. Acá ponemos los servidores con node_exporter y para las máquinas que tienen docker ponemos la dirección con el puerto 8080 para ver los datos de cadvisor.

4) **ping-targets.yml** contiene una lista de IP públicas y privadas de los servidores docker que se van a monitorizar, se pueden agregar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Se utiliza blackbox para hacer los pings.

5) **ssl-docker-targets.yml** contiene una lista de páginas que se van a monitorizar, se pueden agregar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Solo podemos monitorizar la URL, IP pública e IP privada que se monitorizan con **blackbox_exporter**.

En el repositorio hay dos archivos .json que tienen dos dashboard de ejemplo: 
1) Monitoreo Docker.json este dashboard monitorea los certificados ssl de las páginas web que tengamos y la utilización de RAM, CPU, tráfico de red, etc. de nuestro servidor docker.
2) Monitoreo Paginas Web.json monitorea páginas web individuales que estén en algún servidor Linux, al igual que con docker monitorea certificados ssl, RAM, CPU, etc.<br>

Agregue un explorador de archivos llamado [File Browser](https://filebrowser.org/) ya que vamos a necesitar modificar bastante los archivos de configuración esta opción nos da acceso a los archivos de configuración del servidor sin utilizar ftp, ssh, etc. Esto lo comente para no cargarlo por defecto, en el repositorio [Mikiztly/portainer](https://github.com/Mikiztly/portainer) ya está agregado.

<h2>Archivos de Configuración</h2>

<h3 id="blackbox-targets.yml">blackbox-targets.yml:</h3>

Con este archivo carga la lista de sitios que se quieren a monitorizar, en el archivo se pone una línea para monitorear cada página de la siguiente forma:<br>

```shell
<BLACKBOX_EXPORTER_IP_PORT>:_:<MODULO>:_:<ESTADO>:_:<TIPO_IP>:_:<INSTANCIA>
```

Detalles:<br>

**\<BLACKBOX_TARGETS_IP_PORT>:** Se debe poner la IP/URL en donde corre el blackbox, si la página no lo tiene se debe poner la IP/URL de prometheus<br>

**\<MODULO>:** Hay dos módulos:

1) [http_2xx] para monitorizar la URL y certificados ssl
2) [icmp] para verificar por ping que la IP/URL esté activa, tambien sirve para ver IP pública y/o local<br>

**\<ESTADO>:** Sirve para agrupar las páginas, por ejemplo: Producción / Pruebas / Desarrollo

**\<TIPO_IP>:** Otra etiqueta para saber que estoy monitorizando: URL - IP PÚBLICA - IP PRIVADA<br>

**\<INSTANCIA>:** Es el nombre de la INSTANCIA (instance en Grafana) para saber que pagina se monitorea<br>

Ejemplo:
```shell
- targets:
  # Pruebas
  # SRV-calendar-merge
  - 5.24.14.5:9115:_:icmp:_:Calendario:_:Publica:_:SRV-calendar-merge
  - 192.244.0.5:9115:_:icmp:_:Calendario:_:Privada:_:SRV-calendar-merge

  # En Produccion
  # sitio
  - sitio.com.ar:9115:_:http_2xx:_:WEBs:_:URL:_:sitio.com.ar
  - 5.22.40.92:9115:_:icmp:_:WEBs:_:Publica:_:sitio.com.ar
  - 192.243.0.181:9115:_:icmp:_:WEBs:_:Privada:_:sitio.com.ar

  # Workie El Moro
  - workie.sitio.com.ar:9115:_:http_2xx:_:Workie:_:URL:_:workie.sitio.com.ar
  - 5.2.1.6:9115:_:icmp:_:Workie:_:Publica:_:workie.sitio.com.ar
  - 192.244.0.16:9115:_:icmp:_:Workie:_:Privada:_:workie.sitio.com.ar
```

<h3 id="nodes-targets.yml>">nodes-targets.yml:</h3>

Con este archivo carga la lista de servidores Linux que vamos a monitorizar, podemos utilizar dos exporters:

1) node_exporter: para obtener los recursos de una máquina Linux y utiliza el puerto 9100
2) cadvisor: con este exporter se obtienen los recursos utilizados por los contenedores de Docker y utiliza el puerto 8080<br>

Cada línea sirve para monitorizar el servidor y debe ir de la siguiente forma:

```shell
<NODES_TARGETS_IP_PORT>:_:<ESTADO>:_:<INSTANCIA>
```

Detalles:<br>

**\<NODES_TARGETS_IP_PORT>:** Se debe poner la IP/URL en donde corre el node_exporter, osea la del servidor.

**\<ESTADO>:** Sirve para agrupar LOS SERVIDORES, por ejemplo: Linux / Pruebas / Páginas

**\<INSTANCIA>:** Es el nombre de la INSTANCIA (instance en Grafana) para saber que se monitorea

Ejemplo:
```shell
- targets:
  # Pruebas
  - 192.250.4.2:9100:_:Linux:_:soporte.strong.local # node_exporter para monitorear la máquina
  - 192.250.4.2:8080:_:Soporte:_:soporte.strong.local # cadvisor para monitorear docker

  # En Producción
  - node_exporter:9100:_:Linux:_:grafana.strong.local # node_exporter para monitorearla  máquina
  - cadvisor:8080:_:Monitor:_:grafana.strong.local # cadvisor para monitorear docker
  - 192.243.0.181:9100:_:Paginas:_:sitio.com.ar # node_exporter para monitorear la máquina
  - 192.243.0.253:9100:_:Paginas:_:algo.sitio.com.ar # node_exporter para monitorear la máquina

  # Páginas web
  - 192.244.0.10:9200:_:Webs:_:Paginas-WEB # node_exporter para monitorear la máquina
  - 192.244.0.10:8080:_:Webs:_:Paginas-WEB # cadvisor para monitorear docker
```

<h3 id="ping-targets.yml>">ping-targets.yml:</h3>

Con este archivo carga la lista de IP que vamos a monitorizar, en el archivo se pone una línea para monitorear la IP de la siguiente forma:

```shell
<PING_TARGETS_IP_PORT>:_:<MODULO>:_:<ESTADO>:_:<TIPO_IP>:_:<INSTANCIA>
```

Detalles:

**\<PING_TARGETS_IP_PORT>:** Se debe poner la IP en donde corre el blackbox, si la pagina no lo tiene se debe poner la IP de prometheus

**\<MODULO>:** Se utiliza el modulo [icmp] para verificar por ping que la IP esté activa, sirve para ver IP pública y/o local

**\<ESTADO>:** Sirve para agrupar las máquinas, por ejemplo: Producción / Pruebas / Desarrollo

**\<TIPO_IP>:**  Otra etiqueta para saber que estoy monitorizando: URL - IP PÚBLICA - IP PRIVADA

**\<INSTANCIA>** Es el nombre de la INSTANCIA (instance en Grafana) para saber que se monitorea

Esta configuración es parecida a blackbox pero solo hay que usarla para monitorizar la IP publica y/o privada de páginas dentro de docker.<br>

Ejemplo:

```shell
- targets:
  # Pruebas
  - 5.2.40.209:9115:_:icmp:_:WEB:_:PubDock:_:terrallende.com.ar
  - 192.244.0.24:9115:_:icmp:_:WEB:_:PrivDock:_:terrallende.com.ar

  # Páginas web Docker
  - 5.22.14.19:9115:_:icmp:_:Docker:_:PubDock:_:Paginas-WEB
  - 192.244.0.10:9115:_:icmp:_:Docker:_:PrivDock:_:Paginas-WEB
```
<h1 id="Configuración de clientes">Configuración de clientes</h1>

<h2 id="Instalación de node_exporter" >Instalación de node_exporter</h2>

En sistemas basados en Debian debemos bajar el ejecutable desde la [página oficial](https://prometheus.io/download/) de Prometheus, podemos lograrlo yendo a la página y haciendo click derecho en la url del archivo y después en “Copiar enlace”

![descarga_node_exporter](/imagenes/descarga_node_exporter.png)<br>

Con esto podemos armar el comando para descargar la última versión del exporter

```shell
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```

Una vez descargado lo descomprimimos con
```shell
tar xvzf node_exporter-1.8.2.linux-amd64.tar.gz
```

Después cambiamos el propietario y grupo de los archivos descargados y movemos el ejecutable a la carpeta /usr/bin
```shell
chown -R root:root node_exporter-1.8.2.linux-amd64
mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/bin/
```

Ahora creamos un servicio en la carpeta systemd
```shell
vim /lib/systemd/system/node_exporter.service
```

Le pegamos el siguiente código:
```shell
[Unit]
Description=Node Exporter
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Recargamos los servicios
```shell
systemctl daemon-reload
```

Habilitamos el servicio node_exporter
```shell
systemctl enable node_exporter
```

Lo iniciamos
```shell
systemctl start node_exporter
```

Verificamos que esté corriendo sin errores
```shell
systemctl status node_exporter
```

![instala_node_exporter](/imagenes/instala_node_exporter.png)<br>

Si vemos algo parecido es por que el servicio se inició sin errores

<h2 id="Instalación de blackbox_exporter" >Instalación de blackbox_exporter</h2>

En sistemas basados en Debian debemos bajar el ejecutable desde la [página oficial](https://prometheus.io/download/) de Prometheus, podemos lograrlo yendo a la página y haciendo click derecho en la url del archivo y después en “Copiar enlace”

![descarga_blackbox_exporter](/imagenes/descarga_blackbox_exporter.png)<br>

Con esto podemos armar el comando para descargar la última versión del exporter
```shell
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
```

Una vez descargado lo descomprimimos con
```shell
tar xvzf blackbox_exporter-0.25.0.linux-amd64.tar.gz
```

Después cambiamos el propietario y grupo de los archivos descargados y movemos el ejecutable a la carpeta /usr/bin
```shell
chown -R root:root blackbox_exporter-0.25.0.linux-amd64
mv blackbox_exporter-0.25.0.linux-amd64/blackbox_exporter /usr/bin/
```

También creamos la carpeta para guardar el archivo de configuración y lo movemos
```shell
mkdir /etc/prometheus && mv blackbox_exporter-0.25.0.linux-amd64/blackbox.yml /etc/prometheus
```

Ahora creamos un servicio en la carpeta systemd
```shell
vim /lib/systemd/system/blackbox.service
```

Le pegamos el siguiente código:
```shell
[Unit]
Description=Blackbox Exporter Service
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/bin/blackbox_exporter \
  --config.file=/etc/prometheus/blackbox.yml \
  --web.listen-address=":9115"

Restart=always

[Install]
WantedBy=multi-user.target
```

Recargamos los servicios
```shell
systemctl daemon-reload
```

Habilitamos el servicio node_exporter
```shell
systemctl enable blackbox
```

Lo iniciamos
```shell
systemctl start blackbox
```

Verificamos que esté corriendo sin errores
```shell
systemctl status blackbox
```
![instala_blackbox_exporter](/imagenes/instala_blackbox_exporter.png)<br>

Si vemos algo parecido es por que el servicio se inició sin errores

<h2 id="Instalación en Docker>">Instalación en Docker</h2>

Se puede descargar el archivo exporters.yml desde la esta página de Git o la mejor opción es descargarlo en el servidor a monitorizar con:
```shell
wget https://github.com/Mikiztly/grafana/raw/refs/heads/main/exporters.yml
```

Este archivo tiene los cuatro exporters para poder obtener toda la información del servidor, también se puede crear un compose con los exporters que se necesiten. Una vez bajado o creado el archivo compose se puede cargar en docker con el siguiente comando:
```shell
docker-compose -f exporters.yml up -d
```

