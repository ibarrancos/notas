# Proceso de desarrolo y mantenimiento de aplicaciones

Este documento presenta una serie de **puntos abiertos a debate** que tienen como objetivo elaborar una serie de recomendaciones de buenas prácticas que sirvan como marco de referencia para la gestión de proyectos software.

Otras comunidades autónomas han hecho algo similar:

1. El *Gobierno de Canarias*, en su perfil del contrante tienen publicado [AGRIOTP_ManualProveedor_v1_4.pdf](http://www.gobiernodecanarias.org/perfildelcontratante/apipublica/documento.html?documento=128468&anuncio=133460)

2. El *Govern de les Illes Balears* publica también las [recomendaciones técnicas](http://governib.github.io/) a seguir en el desarrollo de aplicaciones en [GitHUB](https://github.com/GovernIB/)

3. Y por supuesto, la Junta de Andalucía en su MArco de DEsarrollo de la Junta de Andalucía: *[MADEJA](http://www.juntadeandalucia.es/servicios/madeja/contenido)* 


En los tres casos encontraremos directrices comunes sobre cómo gestionar el control de versiones y organizar el código en él (ramas/trunk/ tags), cómo construir los artefactos o qué documentación se debe elaborar, y se describe el marco de trabajo y las herramientas que se usarán para ello. No hay que insistir **en la necesidad e importancia de este tipo de recomendaciones** en un entorno que cada vez es más dinámico, hay más subcontratación externa y los equipos cuentan con una alta rotación laboral y su continuidad se limita a meses: *Es necesario definir claramente las reglas del juego para todos y difundirlas*.


Si se leen las recomendaciones de cada una de estas instituciones y se profundiza en documentos similares en Internet, se podrán detectar los siguientes puntos centrales, que son los que sometemos a discusión en este documento:

* La elección de un Sistema de Control de Versiones,
* La gestión y organización de los proyectos en el control de versiones,
* El estilo a usar en la escritura del código fuente,
* La gestión de Tickets asociados al desarrollo y mantenimiento de aplicaciones,
* La construcción del software desde el código fuente,
* La gestión de la calidad del software,
* La gestión de la construcción y despliegue de las aplicaciones,

...y para todo ello, se describen las herramientas que se usarán en cada una de las instituciones.

Este documento analizará cada uno de estos puntos centrales, y realizará una propuesta de aplicación en la CARM, que habrá que debatir internamente, para finalmente poder elaborar ese conjunto de recomendaciones técnicas que nos sirvan de referencia a todos, dentro del conjunto de medidas que se están adoptando para *implantación de Integración Contínua*.


## El Sistema de Control de Versiones (SCM)

Actualmente en la CARM encontramos distintos tipos de software:

* **CVS** en distintas unidades de negocio ([ATRM](cvs.carm.es) o [Agricultura](cvs-agri.carm.es))
* **Subservion** en distintas unidades de negocio ([DGPIT](http://vcs.carm.es), [ITREM](http://subversion.tyo.carm.es) o Educación)
* **GIT** en distintas unidades de negocio ([DGPIT](http://gitlab.carm.es) o [Educación](http://gitlab.edu.carm.es)

Este *collage* de sistemas, retrasa la implantación y adopción de cualquier acción destinada a mejorar la integración contínua:

1. Cada sistema es administrado por diferentes equipos con distino nivel de formación, experiencia y dedicación
2. Cada sistema presenta una serie de facilidades e inconvenientes que favorecen o requieren de otros servicios para la implantación de la integración contínua, lo que deriva en un aumento de la duplicidad de sistemas y de las tareas de administración.
3. Cada sistema tiene su propia gestión de autenticación y autorización independiente y no común,
4. Estos factores junto a la descentralización provocan que las medidas que puedan adoptarse lleven diferentes ritmos de implantación,


Y todos desencadenan en dos problemas:

1. Por un lado, un mismo desarrollador debe adaptar su forma de trabajar, si entrega código para la Consejería de Educación, de si lo hace para la de Agricultura o de Industria (y por supuesto solicitar autorizaciones a equipos diferentes)
2. Por otro lado, las reorganizaciones orgánicas por motivos políticos, obligan a menudo a migrar las aplicaciones de SCM, y una aplicación que antes estaba matenida por la Consejería de Educación pase a ser mantenida por la de Industria (con los consiguientes trabajos de migración y autorizaciones)

... que podríamos resumir, en que se invierte más tiempo en crear los repositorios, migrarlos, administrarlos, gestionar las autorizaciones, adecuarse a los diferentes WorkFolws de trabajo, que en el propio trabajo de programación y desarrollo en sí. Por todo ello es importante:

* Definir **un único SCM** de referencia centralizado en la CARM, con un equipo dedicado a su administración,
* Establecer un **plan de migración** a ese sistema único para el resto de sistemas


### Elección del SCM (=GIT)

Sin entrar en el detalle de las ventajas e incovenientes de cada uno de ellos, sí decir que:

* CVS requiere de demasiada intervención del administrador de sistemas y no tiene soporte a commits atómicos
* Subversion no es tan eficiente para la gestión de ramas y tags como GIT, y sigue necesitando del administrador de sistemas que cree el repositorio y configure los permisos,
* Git no tiene estas limitaciones, y además existen herramientas que integran GIT con la gestión de tickes, api rest, planificación del proyecto y su integración contínua, **todo en uno** ***¿por qué renunciar a ello?***

En la CARM ya se tiene [GitLab-CE](http://gitlab.carm.es) instalado y configurado, desde hace dos años:

* Existen equipos de desarrollo que ya están usándolo
* Está configurado para que cualquier persona que orgánicamente dependa del CRI pueda acceder sin solicitar autorización, con sus credenciales corporativas

Además, se tienen otros proyectos forkeados que provienen del ministerio en [GitHUB](https://github.com/carm-es). 

Por todo ello, **NO parece estar en discusión qué sistema SCM adoptar como único**: ```GitLab``` para repositorios privados y ```GitHub``` para los públicos. El criterio de cuándo un repositorio es público parece que también está claro: *Cuando la aplicación requiera interoperar con otras administraciones públicas del nivel que sean*.


## Construcción y despliegue de las aplicaciones

En el marco actual de trabajo para la integración contínua de aplicaciones en la CARM, se han **adoptado ya de facto ```Maven3``` y ```Gradle```**, y se deja a criterio del responsable del proyecto. La **decisión es acertada**, porque habrá proyectos en los que una herramienta resulte más apropiada que la otra y quien mejor lo sabe, es quien conoce el código.

La publicación de artefactos *(SNAPSHOTS y RELEASEs)* resultantes del proceso de construcción con *Maven/Gradle*, se realiza sobre [NEXUS](http://nexus.carm.es). El proceso está bien consolidado para aplicaciones Java, aunque **no aplica de la misma forma a otros lenguajes como PHP, NodeJS, Oracle FORMS, paquetes de sistemas** ***(RPM, DEB, etc)***

La ejecución de tareas de construcción y despliegue se orquesta desde [JENKINS](http://jenkins.carm.es):

1. Existe una tarea de construcción del proyecto que publica a nexus un SNAPSHOT o RELEASE, y cuando se solicita se puede configurar para elegir la rama que construir.
2. Existe una tarea de despliegue que permite seleccionar el artefacto de NEXUS y el entorno en el que desplegar con las siguientes limitaciones:
    * En el entorno de desarrollo sólo se pueden desplegar SNAPSHOTs
    * En el entorno de producción sólo se pueden desplegar RELEASEs
    * En el entorno de pruebas se puede elegir qué desplegar: si SNAPSHOT o RELEASE

El uso de Jenkins es acertado aunque presenta inconvenientes:

* Este modelo **sólo se está aplicando para aplicaciones Java**: No puedo explicar aquí cuál es el proceso que se sigue para PHP, Oracle Forms u otros, lo desconozco *¿dónde están publicados los artefactos de GLPI? ¿Cómo sé cuándo y qué versión se ha instalado de GLPI?*
* **Requiere de la intervención de una administrador de sistemas que configure la tarea de compilación**, que suele ignorar cómo construir el proyecto,
* La evolución de aspectos relacionados con la integración contínua del proyecto, cómo podrían ser la gestión de la calidad del software,  evaluación de tests, etc dependen que los administradores de sistemas configuren las correspondientes tareas en Jenkins, y vuelve a **limitar a indepencia del desarrollador**
* Hasta la fecha, **Jenkins impone limitaciones serias para adopción de CI** como obligar a que los proyectos sólo puedan construirse con Maven3, *¿qué pasa si mi proyecto ya se construía desde el siglo pasado con ANT o Makefile? ¿en serio que tengo que adpatar mi proyecto sólo porque Jenkins tiene instalado Maven3?*

Estos inconvenientes que presenta Jenkins, se subsanarían todos configurando y usando los Pipelines de GitLab-CI, de manera que delegen las tareas de construcción del proyecto al equipo de desarrollo y el equipo de sistemas sólo se limite a las tareas Jenkins del despliegue. En resumen, **la compilación y publicación de artefactos con GitLab-CI y el despliegue con Jenkins, sin entrar a recomendar la herramienta para la construcción del proyecto**.

Esta decisión permitiría integrar en el mismo flujo de trabajo las aplicaciones desarrolladas en [cualquier lenguaje de programación](https://docs.gitlab.com/ee/ci/examples/), aunque para ello se requiere de la instalación de [gitlab-runers con docker](gitlab-runer.md) en el entorno de la CARM.


## Gestión de Tickets

En la CARM se usa como sistema de [gestión de tickets: GLPI](https://glpi.carm.es). Su uso está muy extendido y consolidado, pero aunque se muestra ideal para la gestión general de incidencias informáticas, es claramente insuficiente para el mantenimiento y desarrollo de aplicaciones informáticas,

* No es lo mismo cambiar un ratón o una pantalla, que implementar una integración con un ministerio,
* No existe ningún tipo de conexión entre los tickets de GLPI y los commits, branches o versiones de la aplicación, si no la establece el desarrollador en los commits (poniendo el link al GLPI) o en los seguimientos indicando links a los trabajos de Jenkins (que se eliminan pasado el tiempo) o al commit, ni tampoco se le ha dicho que lo haga. *Hay desarrolladores que ni siquiera escriben texto en los commits*
* Los parámetros de calidad que se establecen en GLPI son apropiados para la gestión de la atención a usuarios (principalmente orientados a SLA y SLO), pero se muestran insuficientes en el desarrollo y mantenimiento de aplicaciones informáticas.


Existen parámetros de medición de la calidad del software, que no recoge GLPI:
* Bugs en el código
* Vulnerabilidades
* Deuda técnica
* Duplicación de código
* Indice de cobertura de los tests,

... y que repercuten directamente en el número de tickets que registran en GLPI los usuarios y la impresión que tienen estos sobre las aplicaciones: *Se tarda demasiado en arreglar bugs, no se incluyen nuevas demandas, tickets recurrentes, etc*. La gestión de proyectos en GLPI tampoco está consolidada lo suficiente como para evaluar su adecuación a las tareas de desarrollo.


### Git Issues

Tanto GitLab como GitHub incluyen la aplicación ISSUES para la gestión de tickets asociados a un repositorio de código fuente, y dado que el SCM debería ser Git, **aprovechemos esta aplicación** tanto para GitLab como para GitHub.

El Workflow de trabajo está muy bien recodigo en [https://ingenieriadesoftware.es/proceso-desarrollo-github-infografico/
](https://ingenieriadesoftware.es/proceso-desarrollo-github-infografico/)

Por tanto, lo único que estaría abierto a debate es, cómo deberían reportarse y gestionar los issues en los repositorios. Un buen punto de partida podría ser la [sección cuarta de este documento](https://github.com/datosgobar/taller-github-101/blob/master/presentacion_taller_github_101.md#comunidad--gesti%C3%B3n-de-proyectos). 

Para poder elaborar una propuesta en la gestión de etiquetas, resulta inspirador leer: 

* [https://robinpowered.com/blog/best-practice-system-for-organizing-and-tagging-github-issues/](https://robinpowered.com/blog/best-practice-system-for-organizing-and-tagging-github-issues/)
* [https://github.com/moimikey/issues-label-standard/](https://github.com/moimikey/issues-label-standard/)
* [https://docs.saltstack.com/en/latest/topics/development/labels.html](https://docs.saltstack.com/en/latest/topics/development/labels.html)


En general también podemos fijarnos en cómo lo hacen otros [como el desarrollo del propio GitLab](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/CONTRIBUTING.md)


**Este debate debería cerrarse con**:

1. Recomendaciones de lo mínimo que debe recoger un ISSUE
2. Qué etiquetas deben usarse como mínimo en cada proyecto.
3. Cuándo y cómo crear y cerrar un Milestone



## Flujo de trabajo con el control de versiones

Se debe elegir un modelo para el flujo de trabajo: Ramas mínimas de un proyecto y cómo trabajar con ellas. Es muy esclarecedor [leer   https://buddy.works/blog/5-types-of-git-workflows](https://buddy.works/blog/5-types-of-git-workflows) y la **apuesta clara debería ser ```Gitflow```** como justifica [Ana M. del Carmen García Oterino en este post](http://www.javiergarzas.com/2014/05/control-versiones-scrum.html).

Toda la problemática está perfectamente recogida en [https://nvie.com/posts/a-successful-git-branching-model/](https://nvie.com/posts/a-successful-git-branching-model/) y resumida gráficamente en [https://www.slideshare.net/armenta_angel/tcnicas-avanzadas-de-control-de-versiones](https://www.slideshare.net/armenta_angel/tcnicas-avanzadas-de-control-de-versiones), aunque para Subversion lo cual facilita su comprensión.

El resumen que global de todo podríamos encontrarlo [en este artículo](https://delicious-insights.com/fr/articles/git-workflows-conventions/), donde trata todo el proceso completo: Desde cómo escribir los commits, aplicar estilo al código, gestionar los issues a qué documentación mínima debería elaborarse.


**Este debate debería cerrarse con**:

1. Recomendaciones de cuántas ramas como mínimo debe haber por proyecto
2. Cómo deben nombrarse las ramas
3. Cómo deben escribirse los commits.
4. Qué documentos mínimos deben escribirse y qué deben recoger: ```README.md```, ```CONTRIBUTING.md```, ```LICENSE.md```
5. Cuándo deben realizarse MergeRequest y cómo nombrarse.


### Licencia

Todos los proyectos deberían tener su correspondiente licencia.

Aquí una disertación sobre ello: [https://www.heraldo.es/noticias/suplementos/tercer-milenio/itainnova/2017/05/31/aspectos-legales-tener-cuenta-desarrollo-vendo-software-1178653-2121031.html](https://www.heraldo.es/noticias/suplementos/tercer-milenio/itainnova/2017/05/31/aspectos-legales-tener-cuenta-desarrollo-vendo-software-1178653-2121031.html)

Aquí un resumen de las licencias entre las que podemos elegir: [https://choosealicense.com/appendix/](https://choosealicense.com/appendix/)

Aquí [http://repositorio.ual.es/bitstream/handle/10835/2412/Trabajo.pdf?sequence=1&isAllowed=y](http://repositorio.ual.es/bitstream/handle/10835/2412/Trabajo.pdf?sequence=1&isAllowed=y), un trabajo Fin de Grado que habla de la legislación española en materia de licencias de Software: *Página21: 3.1. REGULACIÓN LEGAL DEL SOFTWARE LIBRE EN ESPAÑA*


**Este debate debería cerrarse con**:

1. Recomendaciones de qué licencias deben tener todos los desarrollos que se realicen.






