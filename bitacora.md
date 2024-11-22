Introducción
Este proyecto tiene como objetivo implementar una solución de 
virtualización utilizando Docker, que permita consolidar la infraestructura
 de una organización. La empresa poseía tres servidores tipo torre y adquirió
 un nuevo servidor de mayor capacidad. En este proyecto, se crean tres contenedores
con servicios de Apache, MySQL y Nginx, utilizando archivos Dockerfile para la 
creación de imágenes. Además, se configuran volúmenes mediante RAID 1 y LVM para
 garantizar la persistencia de datos. El proyecto incluye pruebas de funcionamiento
 para verificar la integración y correcto desempeño de los servicios.

Pasos Seguidos

Preparación del entorno
Se utilizó VirtualBox para crear una máquina virtual con Ubuntu 24.04.
Se instalaron herramientas necesarias como Docker, mdadm (para RAID), 
y otros paquetes básicos.

Configuración de discos y RAID
Se añadieron 6 discos virtuales de 2 GB cada uno en VirtualBox.
Se configuraron tres arreglos RAID 1 con dos discos cada uno, usando mdadm.
Se creó un LVM en cada RAID para proporcionar volúmenes a los contenedores.
Creación de contenedores con Docker

Apache: Se configuró un contenedor que sirve una página web estática representando
a la empresa "InfraSoluciones".
MySQL: Se configuró un contenedor con persistencia de datos para 
pruebas de bases de datos.
Nginx: Se configuró como servidor proxy inverso.

Pruebas de funcionamiento
Se realizaron pruebas para validar que cada servicio funcionara correctamente.
Se verificó la persistencia de datos en los volúmenes asociados a los contenedores.


Comandos Utilizados
Configuración del entorno
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io mdadm lvm2
sudo systemctl enable docker && sudo systemctl start docker


Configuración de discos y RAID
Verificación de discos: lsblk

Configuración de RAID:
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
sudo mdadm --create --verbose /dev/md2 --level=1 --raid-devices=2 /dev/sdf /dev/sdg

Creación de LVM:
sudo pvcreate /dev/md0 /dev/md1 /dev/md2
sudo vgcreate vg_containers /dev/md0 /dev/md1 /dev/md2
sudo lvcreate -L 1.5G -n lv_apache vg_containers
sudo lvcreate -L 1.5G -n lv_mysql vg_containers
sudo lvcreate -L 1.5G -n lv_nginx vg_containers


Creación de contenedores con Docker

Apache

Archivo Dockerfile:
FROM httpd:latest
COPY ./index.html /usr/local/apache2/htdocs/

Construcción y ejecución:
docker build -t apache-server ./apache
docker run -d --name apache-container -p 8080:80 -v /mnt/volumes/apache:/usr/local/apache2/htdocs apache-server

Página web:
echo "<html><body><h1>Bienvenido a InfraSoluciones</h1></body></html>" > /mnt/volumes/apache/index.html


MySQL
docker run -d --name mysql-container -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234 -v /mnt/volumes/mysql:/var/lib/mysql mysql:latest

Nginx:

Archivo Dockerfile:
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/

Construcción y ejecución:
docker build -t nginx-server ./nginx
docker run -d --name nginx-container -p 8081:80 -v /mnt/volumes/nginx:/usr/share/nginx/html nginx-server


Conclusión
Este proyecto demostró cómo utilizar virtualización y 
contenedores para consolidar infraestructura. Los servicios 
de Apache, MySQL, y Nginx operaron exitosamente con persistencia
 de datos a través de RAID y LVM, asegurando la disponibilidad y 
redundancia de datos. Se logró integrar cada componente y validar su 
funcionamiento en un entorno realista, siguiendo las mejores prácticas 
de infraestructura computacional.
