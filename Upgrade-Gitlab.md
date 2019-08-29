
## Migracion de GitLab

Hace tiempo instalé Ubuntu 18.04 para probar GitLab. Esta tarde he decido hacer más pruebas con GitLab-CI y me encuentro al ejecutar ```apt-get dist-upgrade```:

```
Preparando para desempaquetar .../gitlab-ce_12.2.1-ce.0_amd64.deb ...
gitlab preinstall: It seems you are upgrading from 10.x version series
gitlab preinstall: to 12.x series. It is recommended to upgrade
gitlab preinstall: to the last minor version in a major version series first before
gitlab preinstall: jumping to the next major version.
gitlab preinstall: Please follow the upgrade documentation at https://docs.gitlab.com/ee/policy/maintenance.html#upgrade-recommendations
gitlab preinstall: and upgrade to 11.11 first.
dpkg: error al procesar el archivo /var/cache/apt/archives/gitlab-ce_12.2.1-ce.0_amd64.deb (--unpack):
 new gitlab-ce package pre-installation script subprocess returned error exit status 1
Se encontraron errores al procesar:
 /var/cache/apt/archives/gitlab-ce_12.2.1-ce.0_amd64.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

Ni recordaba la versión que tenía instalada... [http://gitlab.casa.tecnoquia.com/help](http://gitlab.casa.tecnoquia.com/help), que en ese momento era 10.7.3.
Buscano un poco por internet tropiezo con [Upgrade GitLab from Version 10.7.3 to 11.4.7](https://pikedom.com/upgrade-gitlab-from-version-10-7-3-to-11-4-7/)... ¡¡ justo lo que necesitaba!!

El blog recomienda hacer saltos intermedios antes de actualizarse a la última ```10.7.3 -> 10.8.7 -> 11.4.7```, también porque parace que hay bugs importantes en las versiones 10.x que afectaban al proceso de actualización.


Manos a la obra:

```
# Saltar primero desde 10.7.3 hasta 10.8.7 
apt install gitlab-ce=10.8.7-ce.0

# ufff... tarda la vida
gitlab-ctl restart

# Compruebo: http://gitlab.casa.tecnoquia.com/help

# Ahora saltar desde 10.8.7 hasta 11.4.7
apt install gitlab-ce=11.4.7-ce.0

# ufff... más vida aquí perdia...
gitlab-ctl restart

# Vuelvo a comprobar: http://gitlab.casa.tecnoquia.com/help

# Hay que seguir actualizando... antes de estar a la última
apt list -a gitlab-ce
apt install gitlab-ce=11.11.8-ce.0

# ufff... más vida aquí perdia...
gitlab-ctl restart

#...y ya por fin hasta dentro!
apt-get update
apt-get -y upgrade

```

## Gitlab-runner 

Después de hacer pruebas con GitLab-CI, compruebo que tras estar totalmente actualizado, los Pipelines no funcionan, porque parece que en algún momento GitLab estableció que la conexión debía hacerse por tcp/2375 y no por sockets unix. 

En la web de soporte de Docker explican [cómo configurar el servicio para que escuche tcp/2375 (cifrado) y tcp/2376 (en claro)](https://success.docker.com/article/how-do-i-enable-the-remote-api-for-dockerd)

```
 # En el gitlab-runner

 # Comprobar que el usuario gitlab-runner puede conectar
 sudo -u gitlab-runner -H docker info

 # Sino... meterlo en el grupo
 usermod -aG docker gitlab-runner

 ##
 ## Además, hay que asegurarse que el demonio docker escucha tcp:2375
 ## 
 mkdir /etc/systemd/system/docker.service.d
 
 cat <<EOF >/etc/systemd/system/docker.service.d/startup_options.conf

# /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd://  -H tcp://0.0.0.0:2375 -H tcp://0.0.0.0:2376

EOF

  systemctl daemon-reload
  systemctl restart docker.service

  netstat -putna | grep 237
```
