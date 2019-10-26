# Introducción

Durante los últimos ocho años hemos asistido a profundos cambios en la Administración Regional en relación al uso y explotación de las tecnologías de la información y las comunicaciones (TIC). Estos cambios tienen su origen en la optimización de costes y recursos motivados por los recortes presupuestarios que sufrimos todas las administraciones públicas después de 2009, y caracterizados en diferentes fases:

En una primera fase, se abordó la externalización de toda la infraestructura sobre la que se sustenta nuestra actividad para la prestación de servicios a la sociedad, pasando de infraestructuras on-premise distribuidas por diferentes Centros de Proceso de Datos (CPD) de la CARM con disparidad de características, a un único CPD externalizado (CRISOL) en el que ya todo estaba virtualizado.

En una segunda fase, se realizó una centralización de los recursos y procesos de gestión de la infraestructura para adaptarlos al nuevo escenario y a las limitaciones impuestas por el contrato de licitación con el proveedor del servicio. 

La  publicación  de  las  leyes    39/2015,  de  1  de  octubre,  del  Procedimiento  Administrativo  Común  de  las  Administraciones Públicas y la Ley 40/2015, de 1 de octubre, de Régimen Jurídico del Sector Público,  marcan el comienzo de la tercera fase, y  obligan al desarrollo un Plan Estratégico de Administración Electrónica de la CARM (PAECARM), mediante el cual se debían adaptar nuestras aplicaciones a la tramitación electrónica y, a la obligatoriedad de relacionarnos electrónicamente  con  los  ciudadanos, empresas y el resto de administraciones públicas.  Ello desencadenaría nuevos desarrollos de software de diferentes licitadores, sin disponer de procesos unificados de cómo recepcionar estas entregas ayudando así a aumentar la entropía de la infraestructura, en manos de un proveedor externo cuya continuidad no está garantizada. 

A finales de 2018, asistimos a la última fase de estos cambios, con la adjudicación del contrato de *Servicio de evolución tecnológica de la plataforma de Administración Electrónica de la CARM e inclusión de nuevos servicios en la misma*, por el cual se pretenden transformar los servicios trasversales que actualmente ofrece la plataforma de Administración Electrónica de la CARM a microservicios e integrarlos en PAECARM, para lo cual no se dispone de la infraestructura necesaria y se propone como requisito la adopción de [OpenShift](https://www.openshift.com/) como plataforma de [Kubernetes](https://kubernetes.io/), como ya han empezado a implantar [otras administraciones públicas](https://contrataciondelestado.es/wps/wcm/connect/41e3862a-63b5-40ad-a7de-8ee415857a5e/DOC_CN2018-612136.pdf?MOD=AJPERES).

La confluencia de todos estos hechos y las nuevas demandas de un entorno cambiante, nos lleva a un nuevo panorama en el que la Administración debe ser capaz de adaptarse de manera ágil, proporcionar información y servicios digitales en cualquier momento, en cualquier lugar y por diferentes canales, generar nuevas formas de relación con los ciudadanos e innovar, aprovechándose de las oportunidades que proporcionan las nuevas tecnologías. 

Para ello, venimos realizando un esfuerzo durante los últimos años en adecuar nuestros procesos al modelo de integración continua, y poder así, automatizar las tareas de construcción y análisis del código de las aplicaciones, además de unificar criterios para el desarrollo de aplicaciones mediante la [elaboración de una guía](https://github.com/carm-es/guias/), que introduce una serie de buenas prácticas y modos de trabajo con los que aumentar la eficacia y calidad en la entrega de producto al usuario  Para poder avanzar en la integración de tests en el proceso de desarrollo y despliegue de aplicaciones, ya se ha detectado la necesidad de un nuevo entorno de explotación de los servicios, similar al de producción contra el que ejecutar las pruebas de integración, lo cual no deja de aumentar aún más la entropía de la infraestructura y la demanda de duplicidad de servicios.



# Objetivo

El objetivo de este documento es el de presentar la **estrategia para la adopción de contenedores Dockers en la infraestructura virtual de la CARM**, y con ello conseguir:

* Explotar el potencial de la cultura DevOps, facilitando la integración entre aplicaciones y la agilidad de operaciones, 
* Cambiar el paradigma de desarrollo de aplicaciones, y permitir que los Desarrolladores puedan disponer de un entorno similar al que usará su aplicación en producción y participar de ello,
* Reducir el número de sistemas y la complejidad del mantenimiento de la infraestructura al equipo de Operaciones
* Reducir  el tiempo de *puesta en marcha* y *detección de errores* de los servicios a la vez, que se aumenta la calidad
* Flexibilizar las capacidades de escalabilidad de nuestra infraestructura
* Consolidar la migración a cualquier tipo nube


# Virtualización basada en Dockers

La virtualización es una tecnología que consiste en poder compartir una misma infraestructura harware, por múltiples sistemas operativos que funcionan de forma independiente. Esto tiene un beneficio directo en el ahorro de costes, soluciona  problemas de compatibilidad con el hardware, y permite la clonación y migración de sistemas. Por contra, presenta un rendimiento inferior a lo no virtualizado, limita el hardware que poder usar e introduce riesgos en la seguridad de los sistemas.

Hasta el momento, la infraestructura de la CARM está basada en la **virtualización mediante hipervisor**, que consiste fundamentalmente en virtualizar el hardware físico sobre el que instalar un sistema operativo completo en forma de máquinas virtuales, que hay que mantener actualizado. 
La **virtualización basada en contenedores dockers** va un paso más allá, y virtualiza el sistema operativo proporcionando a las aplicaciones los archivos, las variables y las bibliotecas que necesitan para ejecutarse, maximizando así su portabilidad.

Mientras los hipervisores permiten virtualizar la infraestructura mediante máquinas virtuales, los contenedores virtualizan las aplicaciones utilizando el sistema operativo de su host en lugar de proporcionar el suyo propio.

Los contenedores dockers ofrecen un enfoque alternativo a la Virtualización basada en Hipervisores:

* Benefician a los entornos de nube, ya que permiten a las aplicaciones moverse entre las nubes sin necesidad de trabajo para rehacerlas.
* Minimizan los recursos redundantes que cada instancia virtual necesita, permitiendo al mismo Servidor alojar más contenedores que máquinas virtuales comparables
* Ofrecen una nueva forma de ensamblar aplicaciones.
* Mejoran los tiempos de respuesta, ya que se pueden iniciar y detener mucho más rápido que las máquinas virtuales.

Por contra, no son muy recomendables en entornos que exijan versatilidad e independencia con respecto a la carga de trabajo.

La definición y configuración de estos contenedores dockers se realiza mediante ficheros de texto plano (llamados ```Dockerfile```) que permiten tratar la configuración de la infraestructura como el código fuente de un programa y difuminar los límites entre la escritura de aplicaciones y la creación de los entornos donde se ejecutan, en lo que se ha dado en llamar **infraestructura como código** (*IaC*). Esto permite realizar operaciones sobre las aplicaciones de manera programada y elimina la necesidad de realizar operaciones manuales (configuraciones y actualizaciones), obteniendo mayor velocidad, ahorro de costes y reduciendo el riesgo de las operaciones sobre la infraestructura.

Para poder llevar a cabo estas programaciones sobre la infraestructura se requiere de un software especial que realice las funciones de orquestación, dedicadas a la automatización y administración de los contenedores y su interacción, y de operacionalización, destinadas a la implementación y gestión operativa del ciclo de vida de las aplicaciones dentro de los contenedores.



# Estrategia de adopción

Al definir una estrategia de migración a contenedores dockers de la infraestructura de la CARM, debemos segmentar las aplicaciones por categorías:

1. Aquellas que son fácilmente *"dockerizables"*
2. Aquellas que necesitan ser rediseñadas para poder ser *dockerizadas*
3. Aquellas que tienen un impacto alto en los servicios que ofrecen al ciudadano.
4. Y por último, aquellas que dificilmente podrán ejecutarse en un contenedor docker.

En 2015 el artículo de *David W. Cearley* (Gartner) *“Devise an Effective Cloud Computing Strategy by Answering Five Key Questions"*, que 2017 actualizó *Amazon AWS*, ofrece diferentes estrategias para migrar aplicaciones individuales a la nube:

1. **Rehospedaje**: Cada aplicación se migra tal cual, lo que ofrece las ventajas de la virtualización basada en dockers sin el riesgo ni los costos que conlleva la modificación del código. *GE Oil & Gas*, descubrió que se [ahorraba un 30%](https://aws.amazon.com/es/solutions/case-studies/ge-oil-gas/) de sus costos simplemente al realojar.
2. **Refactorizar**: Esta estrategia implica algún cambio en el diseño de la aplicación, pero no cambios a gran escala en el código de la aplicación. Pretende aprovecharse de la nueva infraestructura y  fomentar la innovación continua con las ventajas que ofrece DevOps
3. **Rediseño**: Modificar la base de código y un cambio en la arquitectura de la aplicación para que esta pueda escalar su rendimiento con docker.
4. **Retirar**:  Al realizar migraciones de infraestructura se suele detectar que hasta un 10 por ciento de la cartera de aplicaciones ya no es útil, y simplemente se pueden desactivar. 
5. **Retener**: por lo general, esto significa “volver a visitar” o no hacer nada (por ahora). Es posible que aún no se esté listo para priorizar una aplicación que se haya actualizado recientemente o que, simplemente no se esté dispuesto a migrar.


**La propuesta para la evolución de nuestra infraestructura a contenedores dockers** consta de tres fases:

1. La primera fase, consiste en **implantar la infraestructura de orquestación necesaria para aplicarla en las aplicaciones que conforman el PAECarm horizontal**, usando la estrategia del *Rediseño*. *Esta fase debería estar acabada en Verano de 2020*.
2. En una segunda fase se abordaría la **migración del resto de aplicaciones fácilmente dokerizables o que tiene un alto impacto en los servicios que se ofrecen al ciudadano**, aplicando estrategias *Refactorizar y Rehospedar*. *Esta fase debería estar acabada en Verano de 2021*.
3. En una tercera fase se abordarían **el resto de aplicaciones difícilmente dockerizables** aplicando estrategia *Retirar y Retener*, esperando simplificar la infraestructura, *para el Verano de 2022*.

Esta propuesta permite **independizarnos del proveedor de Nube y adoptar una cultura DevOps** que implicará cambios en la forma en la que se desarrollan las aplicaciones, se opera con la infraestructura y se gestiona la seguridad... el reto más importante del proyecto. Quedan fuera de alcance,  todas aquellas aplicaciones que implicaran aplicar una estrategia de *Rediseñar*, con la intención de haber desarrollado para entonces la experiencia y capacidades necesarias para abordarlas con garantías de éxito.

Con independencia del cambio cultural que supondría para  la gestión de nuestra infraestructura poder entregar servicios a los ciudadanos más rápido y con mayor calidad,  además se espera que el simple hecho de abordar el proyecto permita **deshacernos de un 10% aproximado de activos y ahorrar al menos un 30% de nuestros costes**, si nos fijamos en los casos de éxito documentados.




# Análisis de la propuesta

Para poder analizar la propuesta de evolución de la infraestructura de la CARM a contenedores dockers, es necesario identificar cuál es nuestra situación actual para poder acometer este proyecto, y reflexionar sobre cuáles son nuestras Fortalezas y Debilidades,  y qué Amenazas y Oportunidades *(DAFO)* se nos presentan al abordarlo .

En general, el análisis en profundidad del proyecto se asemeja a cualquier otro análisis para la migración a la nube, y ya hay suficiente escrito sobre ello:

* [*"Riesgos y amenazas en cloud computing"* del Incibe](https://www.incibe.es/extfrontinteco/img/File/intecocert/EstudiosInformes/cert_inf_riesgos_y_amenazas_en_cloud_computing.pdf)
* [*"Seguridad y resistencia en las nubes de la Administración Pública"* de Enisa](https://www.incibe.es/extfrontinteco/img/File/intecocert/EstudiosInformes/es_governmental_clouds_enisa.pdf)
* [*"Guía para empresas: seguridad y privacidad del cloud computing"* de Inteco](http://www.bono-che.es/resources/guiaempresas_cloudcomputing_accesible.pdf)
* [*"Guidelines on Security and Privacy in Public Cloud Computing "* del NIST](https://www.nist.gov/publications/guidelines-security-and-privacy-public-cloud-computing?pub_id=909494)

Estos artículos, analizan en detalle la migración de la infraestructura TIC a la nube y algunos profundizan en las consecuencias jurídicas de acuerdo a la legislación española de obligado cumplimiento para Administraciones públicas, pero este no es el caso de nuestro proyecto: **Nuestra situación actual es que ya hemos acometido la migración de la infraestructura  a la nube y la tenemos en manos de terceros**, por lo que ese análisis  ya debió realizarse antes de acometer el proyecto de migración a CRISOL: no se analizarán cuestiones como el ahorro de costes energéticos, el marco regulativo que debe aplicarse a la protección de los datos, ni las obligaciones que se contraen con el proveedor del servicio.  La **evolución a dockers es una propuesta para cambiar la forma en la que actualmente gestionamos nuestra nube** con independencia del proveedor.


## Fortalezas
Las fortalezas con las que contamos para poder abordar este proyecto de evolución que hemos identificado serían las siguientes:

* **Optimización de costes**: Se reducen nuestros costes fijos ligados al número de máquinas virtuales y al consumo de disco, debido a que donde ahora se tiene una máquina virtual para ejecutar una aplicación, se pasaría a tener una máquina virtual capaz de ejecutar muchas aplicaciones según nuestras necesidades.
* **Time to market**: Disponer de *Dockerfiles* desde las primeras fases del desarrollo de un servicio y de recursos para escalarlo sin intervención manual, reduciría el tiempo de poner un servicio a disposición de los ciudadanos.
* **Simplificar la administración**: el hecho de modelar toda nuestra infraestructura en ficheros de texto plano que poder reutilizar, simplifica la gestión  de operaciones y se traduce en un ahorro.
* **Pago por uso**: tener nuestra infraestructura dockerizada permite adecuarla rápidamente a nuestras necesidades, y plantear políticas de precios más dinámicas y eficientes con independencia del proveedor de nube.
* **Ubiquidad**: cada vez son más los proveedores que ofrecen servicios de alojamiento para contenedores dockers integrados con nubes híbridas a un precio menor.
* **Tecnología abierta**: Usar estándares abiertos como Docker nos garantiza la compatibilidad con un gran número de soluciones y proveedores mientras que el uso de tecnologías privativas como VMWARE condiciona nuestras posibilidades de futuro.
* **API  WEB  (REST  /  SOAP)**:  A través del orquestador de dockers se hace posible la  provisión de infraestructura mediante la programación de scripts que se ejecutan como respuesta a determinado eventos, y contribuye a disminuir el coste de las operaciones de mantenimiento y aumentar la disponibilidad.
* **Escalado  horizontal**:  Conforme se vaya avanzando en las fases del proyecto, se podrán ir liberando máquinas virtuales dedicadas a aplicaciones y ocupándolas para la ejecución de dockers, que ayudarán a mejorar el rendimiento de lo existente sin aumentar los costes.


## Debilidades
La propuesta cuenta con las siguientes debilidades:

* **Sin modelos de adopción**: Carecemos de modelos en los que basarnos para realizar re-ingeniería de procesos que nos ayuden en la migración, por lo que en muchos casos las soluciones que se adopten contarán con importante *componente ad-hoc*.
* **Sin  modelos  de  riesgo**:  derivado  en  cierta  medida  del  anterior,  carecemos de  modelos  de  riesgo  que  nos  permitan  evaluar  las  decisiones. *Por ello sería muy importante ir de la mano de algún proveedor cualificado*.
* **Dependencia del proveedor**: La decisión del orquestador de docker que usar *(en principio OpenShift de RedHat)*, podría encadenarnos a un proveedor o condicionar la propuesta de migración debido a la solución escogida.
* **Migraciones Inviables**: Podemos encontrar que el número de aplicaciones a migrar sea inferior al estimado y acabar manteniendo dos tipos de aplicaciones: Dockerizadas y no-dockerizadas
* **Soluciones  Ad  hoc**:  Podemos encontrar demasiados casos en los que la solución adoptada  suponga un trabajo personalizado. Esto podría ocultar  cierto nivel de inmadurez en las herramientas escogidas o en nuestra formación. 
* **Escalado  vertical**:  Existirán  aplicaciones  que  no  tengan un  diseño  adecuado  y obligarán a dotarlas de mayor capacidad de cómputo o de  almacenamiento, lo cual podrá suponer un severo inconveniente.
* **Planificación dinámica**: Sin modelos de adopción ni experiencia previa resultará complicado de implementar  en  la práctica. 
* **Programación  distribuida**:  salvo  para  aquellas  aplicaciones  de  simplicidad  extrema,  el resto  de  las  aplicaciones  que  pretenden  sacar  partido  de  todas  las  ventajas que supone el uso de contenedores dockers, requieren ser desarrolladas bajo un enfoque distribuido, en el cual no tenemos ninguna experiencia.

## Oportunidades

* **Estándares**: en estos momentos existe una importante línea de acción en la estandarización  de  elementos  relacionados  con  la  tecnología del Cloud Computing. Ello nos permite adecuar nuestros desarrollos  a  medio  y  largo  plazo  a  los  estándares  en  desarrollo y proteger nuestra inversión,  contribuyendo con nuestras  ideas y experiencia
* **Marketing  global**:  existe  una corriente de opinión  unánime  de  la  conveniencia de  dar este  paso  tecnológico, y la labor de concienciación  está  siendo  realizada  a  todos  los  niveles.  
* **Apoyo  a  las  AAPP**:  Ya hay algunas administraciones  públicas que empiezan a adoptar este tipo de soluciones encontrando financiación en Fondos Feder, lo cual  podría servirnos de inspiración para encontrar financiación que nos permita apoyarnos en proveedores especializados externos.

## Amenazas

El éxito del proyecto se verá amenazado por los siguientes factores:
* **Resistencia  al cambio**:  El cambio cultural en la gestión que se propone encontrará una fuerte resistencia entre nuestros propios técnicos que aún están adaptándose a los cambios que hemos venido sufriendo durante los últimos años.
* Un **cambio de proveedor de CPD** podría alterar las prioridades de trabajo, y alterar la planificación del proyecto.
* **Seguridad**: La virtualización basada en contenedores presenta nuevos retos para la gestión de la seguridad y debido a la falta de un marco legislativo de referencia exclusivo para ello, podría suponer retrasos para la evolución del proyecto.


