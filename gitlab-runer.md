
# GitLab-CI

*[GitLab CI (Continuous Integration)](https://about.gitlab.com/features/gitlab-ci-cd/)* es un **servicio integrado en la interfaz de gestión del proyecto en GitLab, que permite ejecutar accciones después de cada COMMIT**. Estas acciones pueden ser desde la ejecución de test unitarios o de calidad, o la construcción del proyecto, hasta el despliegue de la aplicación en clusters o la creación de máquinas virtuales. Lo interesante de GitLab-CI es que **el resultado de estas ejecuciones quedará registrado junto al commit del desarrollador**, mejorando la trazabilidad del proyecto.

Estas acciones llamadas etapas (*stages*) se definen en un fichero de texto plano en la raíz del repositorio llamado ```.gitlab-ci.yml``` y se ejecutan de forma encadenada (*PipeLine*), de manera que cuando falle una de ellas las siguientes no se ejecutarán.

Esta aproximación tiene sus ventajas: 

1. Relaciona el código fuente de un proyecto con su entorno de integración contínua en una misma herramienta (GitLab)

2. La configuración de la Integración Continua pasa a estar versionada:
    * Fomentando que cada rama tenga su propia configuración.
    * Permite al desarrollador aportar configuraciones, que a veces escapan al técnico de sistemas

3. Integra Docker sin apenas configuración e incluye un Docker registry privado por proyecto.

4. Incorpora un explorador de artefactos para acceder a los resultados de cada etapa

Todo esto se puede conseguir de forma separada con otras herramientas independientes (*```.travis.yml```, Jenkinsfiles,...*) que habría que integrar a base de plugins con GitLab, mientras que GitLab-CI integra todo de forma nativa junto al código fuente, la gestión de Issues, MileStones... se podría decir que *han tomado las mejores funcionalidades de cada herramienta y las han empaquetado en una sola aplicación*.


Para saber más leer: [https://solidgeargroup.com/gitlab_countinuous_integration_intro?lang=es](https://solidgeargroup.com/gitlab_countinuous_integration_intro?lang=es)


A continuación describo los pasos que hay que seguir para configurar nuestro un *[gitlab-runner](https://docs.gitlab.com/runner/)*  compartido en nuestro GitLab.





## Máquina virtual con docker

La idea es disponer de una o más máquinas virtuales configurada para poder ejecutar Docker, registradas como Shared-runner en GitLab.


Para ello, me inspiro en:

* [https://cds.cern.ch/record/2210418/files/Datko.pdf](https://cds.cern.ch/record/2210418/files/Datko.pdf) a partir de la página 29.


### Crear la máquina virtual

Yo creo en mi servidor ```CODE``` una nueva máquina virtual KVM con 1GB RAM , 1CPU y 1 tarjeta de red en mi LAN con **Ubuntu 18.04** a la que configuro:

* Una dirección IP fija y registro en el DNS como ```docker.casa.tecnoquia.com```
* Configuro la zona horaria
```bash
timedatectl set-timezone Europe/Madrid
unlink /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Madrid /etc/localtime
```

Después de instalar el sistema operativo, hay que instalar [docker en Ubuntu 18.04](https://linuxconfig.org/how-to-install-docker-on-ubuntu-18-04-bionic-beaver). El [repositorio para 18.04 en Docker](https://store.docker.com/editions/community/docker-ce-server-ubuntu) aún no está disponible en estable, por lo que decido usar el que viene con el propio sistema operativo: 

```bash

# Actualizar el sistema
apt-get update
apt-get -y dist-upgrade

# Instalar Docker
apt -y install docker.io

# Iniciar y configurar
systemctl start docker
systemctl enable docker

# Comprobar...
docker --version
### Docker version 17.12.1-ce, build 7390fc6

```

### Instalar gitlab-ci-multi-runner

La documentación está en [https://docs.gitlab.com/runner/install/linux-repository.html](https://docs.gitlab.com/runner/install/linux-repository.html)

```bash

# Instalar en dos pasos...
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash

apt-get install gitlab-runner

```

Una vez instalado tendremos que registrarlo en nuestro GitLab. Para ello habrá que ir [http://NUESTRO_GITLAB/admin/runners](http://gitlab.casa.tecnoquia.com/admin/runners) con permisos de administrador y en la sección ```Setup a shared Runner manually```, tendremos las respuestas que más adelante nos preguntará el proceso de registro: 

1. La URL, en *```Specify the following URL during the Runner setup```*
2. El Token, en *```Use the following registration token during setup```*

Localizado esto, ya se puede lanzar el registro del ```gitlab-runner``` mediante:

```bash

# Registrar en nuestro GitLab
gitlab-ci-multi-runner register

```

Se lanzará un asistente que nos realizará las siguientes preguntas:

1. ```Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):```
    * Escribir la URL de nuestro GitLab, en mi caso *http://gitlab.casa.tecnoquia.com/*

2. ```Please enter the gitlab-ci token for this runner:```
    * Escribir el Token

3. ```Please enter the gitlab-ci description for this runner:```
    * Pulsar *ENTER* para aceptar lo que nos propone

4. ```Please enter the gitlab-ci tags for this runner (comma separated):```
    * Pulsar *ENTER* para aceptar lo que nos propone

5. ```Whether to lock the Runner to current project ```[true/false]```:
    * Pulsar *ENTER* para aceptar lo que nos propone

6. ```Please enter the executor: docker-ssh, ssh, docker-ssh+machine, docker, shell, virtualbox, docker+machine, kubernetes, parallels:```
    * Escribir *docker*

7. ```Please enter the default Docker image (e.g. ruby:2.1):```
    * Escribir *ruby:2.1*


Si ahora refrescamos [http://NUESTRO_GITLAB/admin/runners](http://gitlab.casa.tecnoquia.com/admin/runners), ya debe aparecernos el servidor que acabamos de configurar.


### Configurar gitlab-ci-multi-runner

Interesa configurar el fichero ```/etc/gitlab-runner/config.toml``` para añadir la configuración del DNS que debe inyectar de forma predeterminada a los contenedores que ejecute:

```
 dns = ["192.168.2.20","8.8.8.8"]
 dns_search = ["casa.tecnoquia.com"]
```

Las opciones de qué se puede configurar están documentadas en [https://docs.gitlab.com/runner/configuration/advanced-configuration.html](https://docs.gitlab.com/runner/configuration/advanced-configuration.html). 

Esto puede servir para configurar URLs de nuestro entorno como la URL del Nexus, Login, Passwords... a través de variables de entorno que llegan a los contenedores y que los desarrolladores no tienen por qué conocer. 



## Docker-Registry de GitLab

El [docker-registry de GitLab](https://docs.gitlab.com/omnibus/architecture/registry/) es otro servicio integrado en GitLab que ofrece un repositorio de imágenes docker privado, sin necesidad de tener que usar [https://hub.docker.com/](https://hub.docker.com/).

Esto resulta muy atractivo por los siguientes motivos:

1. Podemos **publicar nuestras propias imágenes de docker personalizadas para nuestro organismo** en base a nuestro requerimientos, *por ejemplo si se quiere tener una imagen personalizada de Tomcat o de Apache*

2. Los **repositorios que incluyan ```Dockerfile``` almacenarán la imagen automáticamente** en este registro, *por ejemplo si se quiere preparar un entorno de compilación para maven pre-configurado para nuestro entorno*

3. Los **runners pueden usar imágenes personalizadas** publicadas en el repositorio, que faciliten la construcción de ficheros ```.gitlab-ci.yml```

Para ello habrá que editar el fichero ```/etc/gitlab/gitlab.rb``` de nuestro GitLab, siguiendo los pasos de [http://clusterfrak.com/sysops/app_installs/gitlab_container_registry/](http://clusterfrak.com/sysops/app_installs/gitlab_container_registry/) y habiendo registro en DNS la URL donde publicar este registro *(en mi caso: registry-gitlab.casa.tecnoquia.com)*


### Configuración del registro en los gitlab-ci-multi-runner

Una vez está configurado el registro de docker, debemos decirle al servicio docker de cada uno de los runners que vayamos a usar desde nuestro GitLab cómo puede usarlo.


Para ello,

```bash

# Desde GitLab:
scp /etc/ssl/certs/registry-gitlab.key.crt IP_EQUIPO_DOCKER:/tmp/

# Desde la IP_EQUIPO_DOCKER
mv /tmp/registry-gitlab.key.crt  /usr/local/share/ca-certificates/update-ca-certificates
```

Luego añadir al final del fichero ```/etc/default/docker``` la línea con el nombre DNS de nuestro registro:

```
DOCKER_OPTS="--insecure-registry=registry-gitlab.casa.tecnoquia.com"
```

...y aplicar los cambios:


```bash
# Reiniciar el servicio de docker
systemctl restart docker
```

Por último sólo quedará registrar el equipo en el registro:

```bash
docker login registry-gitlab.casa.tecnoquia.com
```

Nos preguntará:

* ```Username``` escribiremos ```root``` (el administrador de GitLab)
* ```Password``` pondremos la contraseña de acceso del ```root``` a GitLab (ojo, a GitLab NO a la máquina por SSH).




