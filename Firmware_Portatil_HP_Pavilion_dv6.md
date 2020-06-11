# Firmware HP Pavilion dv5

En 2008 compré un [HP Pavilion dv5-1132es](https://icecat.es/es/p/hp/fz519ea/ordenadores-port-tiles-Pavilion+dv5-1132es+Entertainment+Notebook+PC-1791473.html). A los años trabajando con él, empezó a dar fallos:

* Un día, perdió la MAC Address de la tarjeta de red,
* Otro día, perdió la configuración de la BIOS,
* Al tiempo la pila se descargó totalmente.

Pensé que estaba ya inusable, pero parece que se debió a una corrupción de la BIOS. Al tiempo encontré este link https://www.geekslab.it/hp-serial-number-not-found-fix-solution/?lang=en, en el que explicaban como recuperar todo eso:

Los datos de mi portátil son:

* **Serial Number:** CNF8414KGC
* **Notebook Mode:** HP Pavilion dv5 NoteBook PC
* **GUID Number:** *generar aleatorio...*
* **UUID Number:** *generar aleatorio...*
* **SKU Number:** FZ519EA#ABE
* **CTO Localization Code:** ABE
* **Mac Address:** 0D-1E-15-F8-15-16
* **PCID:**  *no lo encuentro!*
* **System Board CT:** 6ACFK02BBWL46K

Lo primero es ejecutar Rufus Portable
```\\COPY\software\01-DRIVERS\BIOS-HP_Pavilion-dv5_portatil_mio```
y formatear un Pendrive auto arrancable con FreeDOS.

Luego descomprimir el contenido de 
```\\COPY\software\01-DRIVERS\BIOS-HP_Pavilion-dv5_portatil_mio\HPDMI.zip```  en el Pendrive.

Después, desmontar con seguridad el Pendrive y arrancar el portátil desde el Pendrive:

```
cd hpdmi
dmifit.bat
```

Rellenar todas las opciones y guardar los cambios.

Aprovechando que tenía que cambiar el disco por uno de 1TB SSD, he intentado averiguar el PCID (que no lo he encontrado claramente), y he recabado la siguiente información:

* **Mac Address EthernetLAN:** 00-21-00-71-B5-73
* **Mac Address EthernetWiFi?:** 00-23-8B-12-BA-85
* **PCID?:** 001B24000141904F 




