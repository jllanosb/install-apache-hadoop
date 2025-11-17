# INSTALACIÓN APACHE HADOOP 3.4.2 EN UBUNTU 24.04

Apache Hadoop es un marco de trabajo (framework) de código abierto diseñado para el almacenamiento distribuido y el procesamiento de grandes volúmenes de datos (Big Data) en clústeres de computadoras. Está construido para escalar desde un solo servidor hasta miles de máquinas, cada una con almacenamiento y capacidad de cómputo local.

## Componentes Principales 
### 1. HDFS (Hadoop Distributed File System) 

#### Propósito: Sistema de archivos distribuido.
Características:
- Tolerante a fallos y altamente disponible.
- Almacena datos en bloques (típicamente de 128 MB en Hadoop 2+).
- Replica los datos (por defecto, 3 copias) en diferentes nodos para evitar pérdidas.

### 2. MapReduce 

#### Propósito: Modelo de programación para procesamiento distribuido.
Fases:
- Map: Procesa los datos de entrada y genera pares clave-valor.
- Reduce: Combina los resultados del map para producir la salida final.

Ideal para: Procesamiento por lotes (batch processing), no para tiempo real.

### 3. YARN (Yet Another Resource Negotiator) 

#### Propósito: Gestiona recursos y programa tareas en el clúster.
Componentes:
- ResourceManager: Coordinador central de recursos.
- NodeManager: Gestiona recursos en cada nodo.
- ApplicationMaster: Supervisa la ejecución de cada aplicación.

### 4. Hadoop Common 

Bibliotecas y utilidades compartidas por todos los módulos.

## Ecosistema de Hadoop 

Además de sus componentes centrales, Hadoop cuenta con un rico ecosistema: 

- Hive: Permite consultar datos con un lenguaje similar a SQL.
- Pig: Lenguaje de alto nivel para flujos de datos.
- HBase: Base de datos NoSQL para acceso en tiempo real.
- Spark: Motor de procesamiento rápido en memoria (más veloz que MapReduce).
- ZooKeeper: Servicio de coordinación distribuida.
- Sqoop: Transfiere datos entre Hadoop y bases de datos relacionales.
- Flume: Recoge y agrega logs o flujos de datos.
- Oozie: Programador de flujos de trabajo.

## Requisitos Previos

- Ubuntu 24.04 instalado (WSL - Virtualizado - Computador con OS)
- Usuario con privilegios sudo
- Mínimo 4GB de RAM
- Al menos 20GB de espacio en disco
- Conexión a Internet

Nota. Si desea usar WSL revisar repositorio ![Instalacion WSL en Windows](https://github.com/jllanosb/Installing-WSL-on-Windows)

# 1. Preparación del sistema operativo (Ubuntu 24.04 Server o Desktop)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ssh rsync openssh-server openssh-client pdsh net-tools vim curl wget tar git -y
```
Verificar IP Servidor
```bash
ifconfig
```
Visualiza algo `inet` como:
```text
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.29.96.93
```
Configurar hostname y hosts
```bash
sudo hostnamectl set-hostname hadoop-master
sudo nano /etc/hosts
```
Agregar al archivo /etc/hosts:

```bash
127.0.0.1 hadoop-master
# 172.29.96.93 del IP Servidor
172.29.96.93 hadoop-master
172.29.96.94 hadoop-worker1
172.29.96.95 hadoop-worker2
# 192.168.1.100 hadoop-master # Servidor Virtualizado Virtualbox, Vmware, otro.
# IP_Publica hadoop-master # VPS, VPC, Cloud
```
# 2. Instalación de Java 11 LTS

Hive productivo 3.1.x / 4 RC estable → Java 11 es estándar.
```bash
sudo apt install openjdk-11-jdk -y
java -version
```

Configurar variables de entorno Java
```bash
sudo nano /etc/environment
```
Agregar en la linea siguiente:
```bash
JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"
```
Actualizar Configuración
```bash
source /etc/environment
```
Verificar instalación
```bash
java -version
javac -version
echo $JAVA_HOME
```
# 3. Crear usuario enterprise dedicado

```bash
sudo groupadd hadoop
sudo useradd -m -g hadoop -s /bin/bash hadoop
sudo passwd hadoop
```
En big data enterprise nunca se trabaja con root para servicios Hadoop.

Configurar permisos sudo para el usuario hadoop
```bash
sudo visudo
```
Agregar la línea al final:
```bash
hadoop ALL=(ALL) NOPASSWD:ALL
```

# 4. Descargar Hadoop 3.4.2 (última versión estable)

Cambiar al usuario hadoop
```bash
sudo su - hadoop
```

Versión a instalar Apache Hadoop 3.4.2 version estable. ![Revisar nuevas Versiones](https://hadoop.apache.org/releases.html)
```bash
cd /tmp
sudo wget https://downloads.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
#sudo wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
sudo mkdir -p /opt/hadoop
sudo tar -xzf hadoop-3.4.2.tar.gz -C /opt/hadoop --strip-components=1
```
Asignar Permisos Propietario
```bash
sudo chown -R hadoop:hadoop /opt/hadoop
```