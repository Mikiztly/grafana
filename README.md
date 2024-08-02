# Monitor Prometheus y Grafana
Son varios archivos para poder montar un servidor con [Prometheus](https://prometheus.io/) y [Grafana](https://grafana.com/) para monitorear equipos que tengamos, ademas estan los exporters y archivos de configuracion para agregar equipos al servidor.<br>
Este archivo lo probe con un servidor manejado con el repositorio [Mikiztly/portainer](https://github.com/Mikiztly/portainer) y ya tiene configurada una red interna, tambien utiliza Nginx-Proxy-Manager para manejar sub-dominios y asi poder tener direcciones web en vez de IP's. Si se va a utilizar solo este repositorio hay que configurar bien la red.<br>
Agregue un explorador de archivos llamado [File Browser](https://filebrowser.org/) ya que vamos a necesitar modificar bastante los archivos de configuracion esta opcion nos da acceso a los archivos de configuracion del servidor sin utilizar ftp, ssh, etc. Esto lo comente para no cargarlo por defecto, en el repositorio [Mikiztly/portainer](https://github.com/Mikiztly/portainer) ya lo tengo agregado.

El archivo **docker-compose.yml** tiene ya configurado un stack donde funcionan Prometheus junto con Grafana, tambien estan los exporters [node-exporter](https://github.com/prometheus/node_exporter) para monitorizar el sistema. [blackbox-exporter](https://github.com/prometheus/blackbox_exporter) que nos permite monitorizar paginas web, [cadvisor](https://github.com/google/cadvisor) que permite ver las estadisticas de docker.

**IMPORTANTE**<br>
Como le agregue el mapeo de volumenes a otro directorio para tener persistencia de datos, en el directorio configurado tienen que existir las liguientes carpetas antes de levantar el stack:

prometheus-config<br>
prometheus-data<br>
grafana-data<br>

Si no existen en el directorio (en mi caso /mnt/docker-data) va a dar error al levantar el stack.

# Configuracion
Hay cinco archivos de configuracion que van en la carpeta **prometheus-config** que definimos en el docker:<br>

1) **prometheus-config.yml** esta con una configuracion personalizada con una configuracion basica para tener el monitoreo local con los 3 modulos, toma la lista de un archivo para obtener una lista de paginas web para monitorizar, Hay que renombrarlo como **prometheus.yml**
2) **blackbox-targets.yml** contiene una lista de paginas que se van a monitorizar, se pueden agreagar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Podemos monitorizar la URL, IP publica e IP privada.
3) **nodes-targets.yml** contiene una lista de servidores que se van a monitorizar, se pueden agreagar con etiquetas. Aca ponemos los servidores con node-exporter y para las maquinas que tienen docker ponemos la direccion con el puerto 8080 para ver los datos de cadvisor.
4) **ssl-docker-targets.yml** contiene una lista de paginas que se van a monitorizar, se pueden agreagar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Tenemos que monitorizar la URL, IP publica e IP privada se monitorizan con otro archivo.
5) **ping-targets.yml** contiene una lista de IP publicas y privadas de los servidores docker que se van a monitorizar, se pueden agreagar con etiquetas como el estado y el tipo de IP utilizada para el monitoreo. Se utiliza blackbox para hacer los pings.

Tambien hay dos arcjivos .json que tienen dos dashboard de ejemplo: **Monitoreo Docker.json** este dashboard monitorea los certificados ssl de las paginas web que tengamos y la utilizacion de RAM, CPU, trafico de red, etc. de nuestro servidor docker. **Monitoreo Paginas Web.json** monitores paginas web individuales que esten en algun servidor linux, al igual que con docker monitores certificados ssl, RAM, CPU, etc.
