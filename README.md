Por aquí colocaré los scripts que corrí...

(Empecé con una máquina desde 0 ya que las máquinas con las que hemos trabajado en clase, la de servidor no deja conectarme por SSH y la de Cliente a veces no conecta, por lo que se creó la máquina desde 0 con los comandos incluidos del VagrantFile para la instalación de todas las herramientas y parte de la configuración, allí coloco los pasos en comentarios)

Vagrantfile: https://www.mediafire.com/file/hnw7qrzkv98ro27/Vagrantfile/file

Una vez alcé la máquina con el init y el up, me conecté por SSH y ejecuté los siguientes comandos:

sudo apt update
sudo apt upgrade -y

Esto para actualizar paquetes y dependencias de Ubuntu

Después de esto iba a crear el usuario para Prometheus, pero recordé que los scripts que provisioné en el vagrantfile ya está todo esto, porque el usuario de prometheus ya existía

sudo useradd --no-create-home --Shell /bin/false prometheus

Intenté creando los directorios 

sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus

Se crearon satisfactoriamente, después, descargué el Prometheus

sudo wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz

sudo tar -xvzf prometheus-2.41.0.linux-amd64.tar.gz

sudo mv prometheus-2.41.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.41.0.linux-amd64/promtool /usr/local/bin/
sudo mv prometheus-2.41.0.linux-amd64/consoles /etc/prometheus
sudo mv prometheus-2.41.0.linux-amd64/console_libraries /etc/prometheus
sudo mv prometheus-2.41.0.linux-amd64/prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

Esto para configurar el archivo de servicio de prometeus

sudo nano /etc/systemd/system/prometheus.service

Este es el contenido

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/

[Install]
WantedBy=multi-user.target

Aquí iniciamos y habilitamos prometeus

sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus

Luego pasamos con Node Exporter

Instalación

sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz


sudo tar -xvzf node_exporter-1.5.0.linux-amd64.tar.gz

Movemos el binario y configuramos permisos

sudo mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter


Configuramos el archivo de servicio de node

sudo nano /etc/systemd/system/node_exporter.service

Contenido

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target


Iniciamos y habilitamos 

sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter

Ahora pasamos con Docker y Docker compose

sudo apt-get update
sudo apt-get install -y docker.io
sudo apt-get install -y docker-compose
sudo systemctl start docker
sudo systemctl enable docker

Clonamos el repo

git clone https://github.com/omondragon/MiniWebApp.git
cd MiniWebApp

Creamos un Docker file con este contenido

FROM python:3.9

WORKDIR /usr/src/app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "./webapp/run.py"]


Creamos un requeriments.txt

nano requirements.txt

Flask==1.1.2
Werkzeug==1.0.1
Jinja2==2.11.3
itsdangerous==1.1.0
MarkupSafe==1.1.1
click==7.1.2
flask_sqlalchemy==2.4.4


Creamos un docker-compose.yml con esto

nano docker-compose.yml

version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"

Construimos y ejecutamos los contenedores

sudo docker-compose up --build

Se ejecuta eso y luego con esta ip se accede por el navegador

http://192.168.50.4:5000

Hubieron varios errores:

El error inicial indicaba que no se encontraba el módulo MySQLdb.
Instalamos mysqlclient y sus dependencias en el contenedor del servidor web Flask.
Se agregó pymysql para usarlo en lugar de mysqlclient
Actualizamos la configuración de SQLAlchemy en config.py para usar pymysql.
Se intentó realizar migraciones de la base de datos usando flask db upgrade, pero se decidió crear la tabla manualmente debido a errores.
Creación manual de la tabla users en MySQL
Se reinició el contenedor de la aplicación web Flask para asegurarse de que la configuración y las conexiones estuvieran actualizadas
Se instaló Python, librerías de Python, se hizo update varias veces, entre otras cosas
Se verificó la funcionalidad de la aplicación accediendo a la interfaz web y probando las operaciones GET y POST.

PARA AWS:

Instala y configura AWS CLI en tu máquina local para facilitar la administración de recursos de AWS.

aws configure

Crear una Instancia EC2

Descagamos en esta máquina sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl start docker
sudo systemctl enable Docker

git clone https://github.com/zh-art/MinWebAppDockerizada.git
cd MinWebAppDockerizada


sudo apt-get install -y docker-compose
sudo docker-compose up --build

Prometheus

http://192.168.50.4:9090

Node Exporter

http://192.168.50.4:9100

Grafana

sudo apt-get install -y gnupg

sudo wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

sudo apt-get update
sudo apt-get install grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

Grafana 

http://192.168.50.4:3000

admin
admin

password

12345

Conections - Data Source prometheus, le colocamos la URL

Create dashboard add visualization prometheus

Dashboard

ID del dashboard de Node Exporter: 1860 (Prometheus Node Exporter Full).

Prometheus

node_cpu_seconds_total
node_memory_MemFree_bytes
node_load1 Execute en la parte de busquedad
