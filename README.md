# Monitor Prometheus y Grafana
Son varios archivos para poder montar un servidor con [Prometheus](https://prometheus.io/) y [Grafana](https://grafana.com/) para monitorear equipos que tengamos, ademas están los exporters [node-exporter](https://github.com/prometheus/node_exporter) para monitorizar el sistema. [blackbox-exporter](https://github.com/prometheus/blackbox_exporter) que nos permite monitorizar paginas web, [cadvisor](https://github.com/google/cadvisor) que permite ver las estadísticas de docker. Para monitorizar los certificados ssl de los contenedores de docker agregue [ssl_exporter](https://github.com/ribbybibby/ssl_exporter) el cual permite monitorizar dentro del contenedor el certificado ssl.<br>
Este archivo lo probe con un servidor manejado con el repositorio [Mikiztly/portainer](https://github.com/Mikiztly/portainer) y ya tiene configurada una red interna, ademas utiliza Nginx-Proxy-Manager para manejar sub-dominios y asi poder tener direcciones web en vez de IP's. Para importar este repositorio en Portainer hay que poner la url del repositorio de la siguiente manera:<br>

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

prometheus/config<br>
prometheus/data<br>
grafana<br>

Si no existen en el directorio (en mi caso /mnt/docker-data) va a dar error al levantar el stack.

# Configuración
Hay cinco archivos de configuración que van en la carpeta **prometheus/config** que definimos en el docker:<br>

1) **prometheus.yml** esta con una configuración personalizada básica para tener el monitoreo local con los 3 módulos, toma la lista de un archivo para obtener una lista de paginas web para monitorizar. Si alguna lista no existe no va a utilizar el exporter, por lo que es util si queremos descartar algún monitor.

2) **blackbox-targets.yml** contiene una lista de paginas que se van a monitorizar, se pueden agregar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Podemos monitorizar la URL, IP publica e IP privada.

3) **nodes-targets.yml** contiene una lista de servidores que se van a monitorizar, se pueden agregar con etiquetas. Aca ponemos los servidores con node-exporter y para las maquinas que tienen docker ponemos la dirección con el puerto 8080 para ver los datos de cadvisor.

4) **ssl-docker-targets.yml** contiene una lista de paginas que se van a monitorizar, se pueden agregar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Solo podemos que monitorizar la URL, IP publica e IP privada se monitorizan con blackbox-targets.yml.

5) **ping-targets.yml** contiene una lista de IP publicas y privadas de los servidores docker que se van a monitorizar, se pueden agregar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Se utiliza blackbox para hacer los pings.

También hay dos archivos .json que tienen dos dashboard de ejemplo: **Monitoreo Docker.json** este dashboard monitorea los certificados ssl de las paginas web que tengamos y la utilización de RAM, CPU, trafico de red, etc. de nuestro servidor docker. **Monitoreo Paginas Web.json** monitorea paginas web individuales que estén en algún servidor Linux, al igual que con docker monitorea certificados ssl, RAM, CPU, etc.

Agregue un explorador de archivos llamado [File Browser](https://filebrowser.org/) ya que vamos a necesitar modificar bastante los archivos de configuración esta opción nos da acceso a los archivos de configuración del servidor sin utilizar ftp, ssh, etc. Esto lo comente para no cargarlo por defecto, en el repositorio [Mikiztly/portainer](https://github.com/Mikiztly/portainer) ya lo tengo agregado.