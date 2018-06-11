# Instalación de SonarQube

SonarQube es una plataforma de código abierto para el análisis de la calidad de código usando diversas herramientas de análisis estático de código fuente como *Checkstyle*, *PMD* o *FindBugs* para obtener métricas que pueden ayudar a mejorar la calidad del código de un programa. 

Permite usarse con [distintos lenguajes de programación](https://www.sonarqube.org/features/multi-languages/) y soporta [cantidad de plugins](https://docs.sonarqube.org/display/PLUG/Plugin+Version+Matrix)

Hay una demo online en [https://next.sonarqube.com/sonarqube/about](https://next.sonarqube.com/sonarqube/about)


Consideraciones previas a tener en cuenta antes de instalar:

1. **Versión**: Deberíamos elegir una [LTS](https://www.sonarqube.org/downloads/), aunque no tengo claro las fechas de publicación de estas versiones.

2. **Requisitos**: La guía oficial de los [requisitos del sistema](https://docs.sonarqube.org/display/SONAR/Requirements) obligan a modificar parámetros del Kernel de Linux, por lo que igual no resulta tan interesante usar [una imagen docker](https://hub.docker.com/_/sonarqube/), con intención de usar frecuentemente SonarQube con nuestro entorno CI

3. **Base de datos**: Aunque soportan Oracle, MySQL y PostGresql, resulta que Oracle tiene bugs reconocidos con el driver JDBC, MySQL debe ser superior a 5.6 *(Debian9 trae MariaDB, que equivale a MySql 5.5)*, por lo que al final casi nos están obligando a usar PostGresQL.

Lo [interesante de SonarQube](https://www.researchgate.net/profile/Gabriel_Spadon/publication/321810844_Teaching_software_quality_via_source_code_inspection_tool/links/5a35892945851532e82f2547/Teaching-software-quality-via-source-code-inspection-tool.pdf?origin=publication_detail) es que podemos integrarlo en nuestro [PipeLine de integración contínua](https://sdos.es/integracion-continua-pasa-por-pipelines/):

* Se **integra con Jenkins y GitLab**
* Se **integra con Maven y Gradle**
* GitLab permite integrarse con SonarQube, de manera que tras los análisis **genere automáticamente issues** en el repositorio y bloquee los *pull request*
* Además de **auditar la calidad del código**, permite **detectar vulnerabilidades** en las dependencias de terceros y **medir la [deuda técnica](https://www.enriquedans.com/2017/06/entendiendo-el-concepto-de-deuda-tecnica.html)** de nuestros desarrollos
* Aporta *consola web* de gestión por proyecto/repositorio





## Instalación sobre máquina virtual

Si la idea es usar SonarQube de forma permanente en nuestro entorno de integración contínua, quizás lo más recomendable sea instalarlo en una máquina virtual independiente, correctamente dimensionada:

* Mínimo 2GB de RAM (recomendado 4GB)
* Mínimo 5GB de espacio en disco (recomendado 20GB)

Hay que tener en cuenta que [SonarQube es básicamente una aplicación Java](https://docs.sonarqube.org/display/SONAR/Architecture+and+Integration):

* ... con un servidor Web embedido
* ... un elastic embebido
* ... un programa que continuamente está accediendo a la base de datos
* ...y una serie de plugins, que al ejecutarse consumen bastante CPU


Para ello, me inspiro en:

* [https://www.vultr.com/docs/how-to-install-sonarqube-on-ubuntu-16-04](https://www.vultr.com/docs/how-to-install-sonarqube-on-ubuntu-16-04)
* [https://www.howtoforge.com/tutorial/how-to-install-sonarqube-on-ubuntu-1604/](https://www.howtoforge.com/tutorial/how-to-install-sonarqube-on-ubuntu-1604/)



### Crear la máquina virtual

Yo creo en mi servidor ```CODE``` una nueva máquina virtual KVM con 1GB RAM , 1CPU y 1 tarjeta de red en mi LAN


```bash

# Cambiar al directorio de la máquina virtual
cd /mnt/nas/VMs/debian-pruebas

# Crear el disco qcow2
qemu-img create -f qcow2 -o preallocation=metadata debian9-so.qcow2 16G

# Fijar permisos
chmod a+w debian9-so.qcow2

```

### Preparar el sistema operativo (Debian9)

Después de instalar el sistema operativo:

```bash

# Actualizar el sistema
apt-get update
apt-get -y dist-upgrade

# Instalar lo que vamos a necesitar
apt-get install -y vim unzip net-tools


# Editar /etc/ssh/sshd_config para añadir 'PermitRootLogin yes' 

# ...y reiniciar el servicio...
systemctl restart sshd

```

...bueno, y todo lo que sea menester (ipv6, systemctl, fijar la ip, agentes vmtools, etc)



### Instalar Java8 de Oracle

Añadir el ```source.list``` para ```apt```:

```bash
cat <<EOF >/etc/apt/sources.list.d/java-8-debian.list

# Java8 de Oracle
deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
EOF

```

Ejecutar los siguientes comandos, para poder importar las claves del repositorio:

```bash
apt-get install -y dirmngr
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886
```

Instalar...

```bash
apt-get update
apt-get install -y oracle-java8-installer
apt-get install -y oracle-java8-set-default
```


### Preparar el sistema operativo

Crear el usuario. *Elastic si corre [como ```root``` no arrancará](https://michalwegrzyn.wordpress.com/2016/07/14/do-not-run-sonar-as-root/)*


```bash

# Crear el grupo
groupadd sonar

# Crear el usuario y su directorio home
useradd -c "Sonar System User" -d /opt/sonar -g sonar -s /bin/bash sonarqube

# Darle permisos...
mkdir -p /opt/sonar
chown -R sonarqube:sonar /opt/sonar

```

Añadir parámetros a ```sysctl.conf```:

```bash
cat <<EOF >/etc/sysctl.d/99-sonarqube.conf

# Requisitos de sonar
vm.max_map_count=262144
fs.file-max=65536

EOF

```

Aumentar los ```ulimit``` como recomienda la [guía de requisitos](https://docs.sonarqube.org/display/SONAR/Requirements#Requirements-Platformnotes):


```bash
cat <<EOF >>/etc/security/limits.conf 

# Para sonar...
sonarqube   -   nofile   65536
sonarqube   -   nproc    2048

EOF

```

Reiniciar y comprobar...

```bash
reboot

sysctl vm.max_map_count
sysctl fs.file-max

su - sonarqube
ulimit -n
ulimit -u
```


### Instalar el servidor de base de datos

Directamente, instalar postgres de Debian9 y configurar para que arranque con el sistema...

```bash
apt-get -y install postgresql postgresql-contrib

systemctl start postgresql
systemctl enable postgresql
```

Crear el usuario de base de datos que usará SonarQube:

```bash
su - postgres
createuser sonar
```

Ahora, entrar en la consola de postgres (```psql```) y crear la base de datos y la contraseña del usuario:

```sql
ALTER USER sonar WITH ENCRYPTED password 'SuperSecret';
CREATE DATABASE sonar OWNER sonar;
\q
```

Igual interesa poder [conectar a postgres desde cualquier máquina de la red](https://blog.bigbinary.com/2016/01/23/configure-postgresql-to-allow-remote-connection.html), yo lo haré por que luego quiero probar con Docker (sin usar ```compose``` para la BD).


Aplicar las siguientes diferencias en ```/etc/postgresql/9.6/main/postgresql.conf```:

```diff
59c59
< #listen_addresses = 'localhost'		# what IP address(es) to listen on;
---
> listen_addresses = '*'			# what IP address(es) to listen on;
```

... y en ```/etc/postgresql/9.6/main/pg_hba.conf```:

```diff
99a100,103
> 
> # Permitir conectar con psql desde cualquier equipo
> host    all             all              0.0.0.0/0              md5
> host    all             all              ::/0                   md5
```

Reiniciar el servicio...

```bash
systemctl restart postgresql
```

### Instalar SonarQube LTS

Ya toca instalar el [software por fin](https://www.sonarqube.org/downloads/). Usaremos la [versión 6.7](https://www.sonarqube.org/sonarqube-6-7-lts/) que es la LTS en el momento de escribir estas líneas. 


```bash

wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-6.7.4.zip
unzip sonarqube-6.7.4.zip  -d /opt

chown -R sonarqube:sonar /opt/sonarqube-6.7.4
rm -fr /opt/sonar
ln -s /opt/sonarqube-6.7.4  /opt/sonar

cp ~root/.bashrc  ~sonarqube/
cp ~root/.profile ~sonarqube/
```

Configurar la conexión a base de datos, aplicando las siguientes diferencias en el fichero ```~sonarqube/conf/sonar.properties``` 

```diff
16,17c16,17
< #sonar.jdbc.username=
< #sonar.jdbc.password=
---
> sonar.jdbc.username=sonar
> sonar.jdbc.password=SuperSecret
39c39
< #sonar.jdbc.url=jdbc:postgresql://localhost/sonar
---
> sonar.jdbc.url=jdbc:postgresql://localhost/sonar
105c105
< #sonar.web.host=0.0.0.0
---
> sonar.web.host=0.0.0.0
```

Decirle a sonar que no debe iniciar como ```root```, aplicando las siguientes diferencias en el fichero ```~sonarqube/bin/linux-x86-64/sonar.sh```

```diff
48c48
< #RUN_AS_USER=
---
> RUN_AS_USER=sonarqube
```

Probar a arrancar el servicio, de forma manual y como ```root```:

```bash
/opt/sonar/bin/linux-x86-64/sonar.sh start
```
Los logs están en ```~sonarqube/logs/```:

* ```sonar.log``` los logs del servicio
* ```es.log``` los logs de elastic
* ```web.log``` los logs de la aplicación web


Crear el script de servicio:

```bash

cat <<EOF >/etc/systemd/system/sonar.service

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonar/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonar/bin/linux-x86-64/sonar.sh stop

User=root
Group=root
Restart=always

[Install]
WantedBy=multi-user.target

EOF

```

Configurar para que se inicie con la máquina:

```bash
systemctl start  sonar
systemctl enable sonar
systemctl status sonar

```

Los scripts estos que he copiado por internet para ```systemctl``` no funcionan bien :( ... sólo funcionan los scripts:

```bash
/opt/sonar/bin/linux-x86-64/sonar.sh start
/opt/sonar/bin/linux-x86-64/sonar.sh stop
/opt/sonar/bin/linux-x86-64/sonar.sh status
```


Después de arrancar se podrá comprobar como los procesos Java empiezan a comer recursos, sobre todo CPU.

Probaremos a acceder a la consola desde un navegador [http://IP_EQUIPO:9000](http://IP_EQUIPO:9000) y autenticarnos inicialmente como ```admin``` y contraseña ```admin```.

Luego ir a la sección ```Administration -> Marketplace``` e instalar los siguientes plugins (pedirá reiniciar varias veces):

* AEM Rules for SonarQube
* Apigee
* Checkstyle
* ClearCase
* Code Smells
* Findbugs
* GitLab
* Issue resolver
* PMD
* SVG Badges
* SoftVis3D Sonar plugin
* SonarJS
* SonarJava
* SonarPHP
* SonarQube :: Plugins :: SCM :: Git
* SonarQube :: Plugins :: SCM :: SVN
* Sonargraph
* Sonargraph Integration
* SonarWeb
* Spanish Pack
* Xanitizer
* jDepend

Con todos estos plugins vamos a poder analizar diferentes aspectos de la calidad del código fuente de un proyecto:

* **Bugs**
* **Vulnerabilidades**, en cuanto a construcciones de código inseguras, según las [recomendaciones OWASP y CWE](https://docs.sonarqube.org/display/SONAR/Security-related+rules)
* **Deuda técnica**, como una estimación del tiempo que debemos invertir en programación para solucionar errores y deficiencias
* **Cobertura** de los tests (0% es que no hay tests, mientras 100% es que los tests elaborados comprueban todo el código)
* Cantidad de código **Duplicado** en nuestro proyecto

Aparte de estos plugins, se echa de menos el [sonar-dependency-check-plugin](https://www.owasp.org/index.php/OWASP_Dependency_Check), que se encarga de escanear las dependencias de nuestro proyecto y detectar fallos de seguridad en componentes de terceros. Parece, por lo que he leido, que esto venía en la versión 5.X, pero no se incluye en la versión 6.X.

Para instalarlo seguimos las instrucciones de [https://github.com/stevespringett/dependency-check-sonar-plugin](https://github.com/stevespringett/dependency-check-sonar-plugin). Lo primero es descargar el plugin para sonar:

```bash
cd ~sonarqube/extensions/plugins
wget https://github.com/stevespringett/dependency-check-sonar-plugin/releases/download/1.1.0/sonar-dependency-check-plugin-1.1.0.jar

/opt/sonar/bin/linux-x86-64/sonar.sh stop
/opt/sonar/bin/linux-x86-64/sonar.sh start

```

En el proyecto en GitHub hay ejemplos de [cómo usarlo con maven](https://github.com/stevespringett/dependency-check-sonar-plugin/tree/master/examples)


## Uso de Sonar

Una vez ya se tiene instalado SonarQube, lo siguiente será lanzar un análisis contra alguno de nuestros proyectos.

### Desde Maven

[Integrar Sonar con Maven](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Maven) está documentado en la página oficial. Lo primero es añadir un nuevo perfil en ```$M2_HOME/conf/settings.xml```:

```xml
<profile>
   <id>sonar</id>
   <activation>
     <activeByDefault>true</activeByDefault>
   </activation>
   <properties>
     <sonar.host.url>http://IP_EQUIPO:9000</sonar.host.url>
   </properties>
</profile>

```

Además habrá que configurar nuestro proyecto, en la sección ```<plugins>```, para añadir los plugins de sonar

```xml
<plugin>
   <groupId>org.sonarsource.scanner.maven</groupId>
   <artifactId>sonar-maven-plugin</artifactId>
   <version>3.4.0.905</version>
</plugin>
<plugin>
   <groupId>org.owasp</groupId>
   <artifactId>dependency-check-maven</artifactId>
   <version>1.4.5</version>
   <configuration>
     <format>XML</format>
     <outputDirectory>${project.build.directory}/dependency-check</outputDirectory>
   </configuration>
</plugin>
```

...y por último ejecutar maven:

```bash

# Construir nuestro proyecto
mvn clean install

# Revisar las dependencias de terceros
mvn dependency-check:check

# Analizar con sonar
mvn sonar:sonar
```

Una vez que acabe maven, generará diferentes ficheros xml en el directorio ```target``` del proyecto que publicará en el servidor Sonar, y este, cuando los reciba, se pondrá a analizarlos para generar el informe, que podremos consultar en la web de Sonar

### Integración con Jenkins

Esto mismo que hacemos desde consola con maven, puede invocarse desde Jenkins a través de un plugin *(similar al plugin de maven-release)*

* [https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins)

* [https://lasithapetthawadu.wordpress.com/2014/05/03/configure-jenkins-with-sonarqube-for-static-code-analysis-and-integration/](https://lasithapetthawadu.wordpress.com/2014/05/03/configure-jenkins-with-sonarqube-for-static-code-analysis-and-integration/
)

* [http://www.tothenew.com/blog/integrating-sonarqube-with-jenkins/](http://www.tothenew.com/blog/integrating-sonarqube-with-jenkins/)


Claro, debe ser el usuario de Jenkins quien lo ejecute, para sonarqube realice el análisis: Si no lo hace no habrá reporte de la calidad de su software.


