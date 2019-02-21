# Integración de hive con hadoop 3.0

 A continuación se presentan los pasos para descargar e integrar hive con hadoop hdfs.
[![N|Solid](https://i.ibb.co/3R24T2w/hive0.png)](https://nodesource.com/products/nsolid)

  **1:** Ir a la pagina **https://hive.apache.org/** y descargar la ultima versión.
 
[![Build Status](https://i.ibb.co/JCQqmC6/hive-Download.png)](https://travis-ci.org/joemccann/dillinger)

Abrir una consola de terminal y descomprimir el .tar.gz que acabamos de descargar 
```sh
arturo@debian$ ls -l
-rw-r--r-- 1 arturo arturo  294M jul 21  2018 apache-hive-3.0.0-bin.tar.gz
arturo@debian$ tar -xvf apache-hive-3.0.0-bin.tar.gz
apache-hive-3.0.0-bin/conf/llap-cli-log4j2.properties.template
apache-hive-3.0.0-bin/conf/parquet-logging.properties
apache-hive-3.0.0-bin/hcatalog/share/doc/hcatalog/README.txt
...
```
Logearse como root y crear el directorio hive en la ruta **/opt/**
```sh
arturo@debian$ sudo su
password: *****
# Crear el directorio hive
root@debian$ mkdir /opt/hive
# Cambiar el propietario concediendo privilegios a mi usuario
root@debian$ chown -R arturo:arturo /opt/hive/
root@debian$ exit # Salir de la sesion de root
# Mover todo el contenido de hive a /opt/hive ya que ahora debemos  tener permisos de escribir en la carpeta /opt/hive
arturo@debian$ mv /home/arturo/Descargas/apache-hive-3.0.0-bin/* /opt/hive
```
**2: Configuración de apache hive**
Es necesario configurar la variable de entorno **HIVE_HOME**, y a continuación debemos de actualizar la variable de entorn **PATH**  para apuntar a los binarios de hive. 

Editar el fichero **.bashrc** el cual se encuentra en /home/**usuario**, en mi caso es **/home/arturo/.bashrc** y agregar las siguientes lineas

```sh
# Agregar estas dos lineas al fichero .bashrc
export HIVE_HOME=/opt/hive
# Para este caso solo debemos agregar la linea: /opt/hive/bin al final de la linea
export PATH=$PATH:/opt/hadoop/sbin:/opt/hadoop/bin/:/opt/hive/bin
```
Para revisar si la configuración esta correcta nos queda abrir un terminal y escribir lo siguiente

```sh
arturo@debian$ hive --version
```
**3: Configurar el Metastore de hive**
 El metastored de hive es un conjunto de directorios(metadatos) en **HDFS** el cual nos indica aspectos importantes de como esta configurado hive en el cluster 
 
```sh
# Es neceario arrancar hadoop para poder crear los siguientes directorios
arturo@debian$ start-dfs.sh #Iniciamos el sistema de ficheros HDFS
Starting namenodes on [hadoop]
Starting datanodes
Starting secondary namenodes [hadoop]
arturo@debian$ start-yarn.sh #Iniciamos el gestor de  procesos YARN
Starting resourcemanager
Starting nodemanagers
```
Revisamos que todos  los procesos estén en ejecución. 

```sh
arturo@debian$ jps
8000 SecondaryNameNode
7808 DataNode
8308 ResourceManager
7702 NameNode
8439 NodeManager
8841 Jps
```
Por ultimo creamos los directorios de **hive en hdfs** 
```sh
arturo@debian$ hadoop fs -mkdir /arturo/
arturo@debian$ hadoop fs -mkdir /arturo/hive
arturo@debian$ hadoop fs -mkdir /arturo/hive/warehouse
arturo@debian$ hadoop fs -mkdir /tmp
```
 **4: Configurar el fichero hive-env.sh.template**
 En **/opt/hive/conf** existe un fichero llamado **hive-env.sh.template** hay que renombrarlo para poder activar su configuración de la siguiente manera **hive-env.sh**, posteriormente agregamos las siguientes líneas 
 
 ```sh
 # Renombrar el fichero
 arturo@debian$ mv /opt/hive/conf/hive-env.sh.template /opt/hive/conf/hive-env.sh
 # Abrir el fichero y agregar las siguientes lineas
 arturo@debian$ vim  /opt/hive/conf/hive-env.sh
export HADOOP_HOME=/opt/hadoop/ # Directorio donde se encuentra la instalación de hadoop
export HADOOP_HEAPSIZE=512 # Tamaño de la pila
export HIVE_CONF_DIR=/opt/hive/conf/
...
# Guardar cambios
 ```
  **4: Cargar la base de datos(definición de metadatos) que viene por defecto(derby)**
  
  Situarse en el directorio donde tenemos instalado hive, en mi caso **/opt/hive**  y escribir el siguiente comando: 
 ```sh
 arturo@debian$ ./bin/schematool -initSchema -dbType derby
 SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:	 jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :	 org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:	 APP
Starting metastore schema initialization to 3.0.0
Initialization script hive-schema-3.0.0.derby.sql
 ```
 Y concluimos con la integración de hive, es recomendable usar hive sobre el motor Tez ya que brinda mas rapidez en la ejecución de consultas, hoy en día existen dos ramas en el desarrollo de hive ya que por una parte se sigue desarrollando hive sobre map-reduce  y por otra parte se abre paso al desarrollo de hive pero ejecutando las consultas sobre el motor Tez. 
 
License
----

GPL

