# Grafana
Son varios archivos para poder montar un servidor con [Prometheus](https://prometheus.io/) y [Grafana](https://grafana.com/) para monitorear equipos que tengamos, ademas estan los exporters y archivos de configuracion para agregar equipos al servidor.<br>
Este archivo lo probe con un servidor manejado con el repositorio [Mikiztly/portainer](https://github.com/Mikiztly/portainer) y ya tiene configurada una red interna, tambien utiliza Nginx-Proxy-Manager para manejar sub-dominios y asi poder tener direcciones web en vez de IP's. Si se va a utilizar solo este repositorio hay que configurar bien la red

**monitoreo.yml**<br>
Se puede utilizar en la consola:<br>
wget -O docker-compose.yml https://github.com/Mikiztly/docker/raw/main/compose/monitoreo.yml

En este archivo esta configurado un stack donde funcionan Prometheus junto con Grafana, tambien estan los exporters [node-exporter](https://github.com/prometheus/node_exporter) para monitorizar el sistema. [blackbox-exporter](https://github.com/prometheus/blackbox_exporter) que nos permite monitorizar paginas web, [cadvisor](https://github.com/google/cadvisor) que permite ver las estadisticas de docker.

**IMPORTANTE**<br>
Como le agregue el mapeo de volumenes a otro directorio para tener persistencia de datos, en el directorio configurado tienen que existir las liguientes carpetas antes de levantar el stack:

prometheus-config<br>
prometheus-data<br>
grafana-data<br>

Si no existen en el directorio (en mi caso /mnt/docker-data) va a dar error al levantar el stack.
