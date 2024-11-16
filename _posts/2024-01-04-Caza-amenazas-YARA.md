---
layout: single
title: Caza de amenazas con YARA
excerpt: "Esta sala se centra en el uso de YARA para la caza de amenazas."
date: 2024-11-12
classes: wide
header:
  teaser: https://tryhackme.4kiing.net/assets/images/Yara/Logo.png
  teaser_home_page: true
categories:
  - TryHackme
tags:
  - Tutorial
---


# Introducción
Esta sala tiene como objetivo demostrar una aplicación activa de la caza de amenazas con un enfoque específico en el uso de las *reglas YARA* para buscar `indicadores de compromiso (IOC)` relacionados con el malware. Utilizaremos un escenario realista como el cable rojo en toda esta habitación.

```text
IOC:

    El indicador de compromiso es un artefacto forense observado en una red o en un sistema operativo que, 
con alta confianza, indica una intrusión informática.
```

## Objetivos de aprendizaje
- Búsqueda de información procesable que se pueda utilizar para buscar amenazas
- Instalación de YARA
- Creación de una regla YARA
- Implementación de una regla YARA

## Prerrequisitos
- Se recomienda haber completado la sala de [Threat Hunting](https://tryhackme.com/r/room/introductiontothreathunting). Esa sala incluye múltiples conceptos y terminologías utilizadas en toda nuestra sala actual.
- Comprensión básica de los conceptos de seguridad, incluidos, entre otros, [Cyber Kill Chain](https://tryhackme.com/r/room/cyberkillchainzmt), TTP, Indicator of Compromise, Hashes y APT.
- Conocimientos básicos sobre el uso de la línea de comandos de Windows y `PowerShell`.
- Conocimientos básicos de los tipos de datos y la codificación.

```text
PowerShell:
    
    PowerShell es un programa de automatización de tareas y administración de configuración de Microsoft, 
que consta de un shell de línea de comandos y el lenguaje de scripting asociado.
```

**Dislaimer:** Utilizaremos un escenario real como cable rojo a lo largo de esta sala. Todas las direcciones URL y los archivos a los que se hace referencia son maliciosos y no deben abrirse fuera de un entorno aislado.

### Responda las siguientes preguntas
>- ¿Estás listo para cazar malware?
<hr>

# Descripción del escenario
Nuestro equipo de caza de amenazas forma parte del CSIRT nacional de Bélgica. Trabajamos en estrecha colaboración con los equipos de inteligencia de amenazas cibernéticas y respuesta a incidentes para adelantarnos a las numerosas amenazas dirigidas a nuestros constituyentes. Una categoría principal de nuestros electores son los partidos políticos.

Nuestro equipo de inteligencia de amenazas cibernéticas ha recogido un interesante [artículo](https://cloud.google.com/blog/topics/threat-intelligence/apt29-wineloader-german-political-parties) del blog de *inteligencia de amenazas Mandiant* sobre un ataque cibernético dirigido a un partido político alemán. Han extraído toda la información relevante y la han pasado a nuestro equipo. Analizaremos esta información y buscaremos oportunidades para cazar la amenaza descrita en el artículo. A continuación se muestra la información extraída por el equipo de inteligencia de amenazas cibernéticas. Han estructurado la información utilizando el modelo del diamante.

## Inteligencia de amenazas proporcionada

```text
Nota:

      Todas las URL y los archivos a los que se hace referencia a continuación son reales y maliciosos. No los abra fuera de un entorno aislado.
```

- `Adversario`: APT29
- `Víctima`: Partido político alemán

**Capacidades utilizadas (TTP)**

|                     Identificación                               |                        Técnica                               |
|------------------------------------------------------------------|--------------------------------------------------------------|
| [T1543.003](https://attack.mitre.org/techniques/T1543/003/)      | Crear o modificar proceso del sistema: servicio de Windows   |
| [T1012](https://attack.mitre.org/techniques/T1012/)              | Registro de consultas                                        |
| [T1082](https://attack.mitre.org/techniques/T1082/)              | Descubrimiento de información del sistema                    |
| [T1134](https://attack.mitre.org/techniques/T1134/)              | Manipulación de tokens de acceso                             |
| [T1057](https://attack.mitre.org/techniques/T1057/)              | Descubrimiento de procesos                                   |
| [T1007](https://attack.mitre.org/techniques/T1007/)              | Detección de servicios del sistema                           |
| [T1027](https://attack.mitre.org/techniques/T1027/)              | Archivos o información ofuscados                             |
| [T1070.004](https://attack.mitre.org/techniques/T1070/004/)      | Eliminación de indicadores: Eliminación de archivos          |
| [T1055.003](https://attack.mitre.org/techniques/T1055/003/)      | Inyección de procesos: secuestro de ejecución de subprocesos |
| [T1083](https://attack.mitre.org/techniques/T1083/)              | Detección de archivos y directorios                          |


### Infraestructura/IOC

```text
Message Digest 5 (MD5)

      Es una función hash criptográfica que toma cualquier entrada y genera un número hexadecimal de 128 bits. La salida de una función hash MD5 se denomina resumen.
  Los resúmenes MD5 se utilizan a menudo para verificar la integridad de archivos o datos; sin embargo, MD5 ya no se considera seguro y no debe usarse para 
  aplicaciones confidenciales.
```

- `Invite.pdf (MD5: fb6323c19d3399ba94ecd391f7e35a9c)`
  - Segundo documento de señuelo PDF con temática de la CDU
  - Escrito en LibreOffice 6.4 por defecto por el usuario **"Writer"**
  - Los metadatos documentan el PDF en idioma *en-GB*
  - Enlaces a https://waterforvoiceless[.]org/invite.php

- `invite.php (MD5: 7a465344a58a6c67d5a733a815ef4cb7)`
  - Archivo zip que contiene **ROOTSAW**
  - Descargado de https://waterforvoiceless[.]org/invite.php
  - Ejecuta *efafcd00b9157b4146506bd381326f39*

- `Invite.hta (MD5: efafcd00b9157b4146506bd381326f39)`
  - Descargador de ROOTSAW que contiene **código ofuscado**
  - Descargas desde https://waterforvoiceless[.]org/util.php
  - Extractos *44ce4b785d1795b71cee9f77db6ffe1b*
  - Ejecuta *f32c04ad97fa25752f9488781853f0ea*

- `invite.txt (MD5: 44ce4b785d1795b71cee9f77db6ffe1b)`
  - Archivo de **certificado malintencionado**, extraído con Windows Certutil
  - Ejecutado desde *efafcd00b9157b4146506bd381326f39*
  - Descargado de https://waterforvoiceless[.]org/util.php

- `invite.zip (MD5: 5928907c41368d6e87dc3e4e4be30e42)`
  - Zip malicioso que contiene **WINELOADER**
  - Extraído de *44ce4b785d1795b71cee9f77db6ffe1b*
  - Contiene *e017bfc36e387e8c3e7a338782805dde*
  - Contiene *f32c04ad97fa25752f9488781853f0ea*

- `sqldumper.exe (MD5: f32c04ad97fa25752f9488781853f0ea)`
  - Archivo legítimo de Microsoft Sqldumper utilizado para la carga lateral (side-loading)

- `vcruntime140.dll (MD5: 8bd528d2b828c9289d9063eba2dc6aa0)`
  - Descargador de **WINELOADER**
  - Se comunica con https://siestakeying[.]com/auth.php

- `Vcruntime140.dll (MD5: e017bfc36e387e8c3e7a338782805dde)`
  - Descargador de **WINELOADER**
  - Se comunica con https://siestakeying[.]com/auth.php

### Detecciones

```text
rule M_APT_Dropper_Rootsaw_Obfuscated
{ 
    meta: 
        author = "Mandiant" 
        disclaimer = "This rule is meant for hunting and is not tested to run in a production environment." 
        description = "Detects obfuscated ROOTSAW payloads" 

    strings: 
        $ = "function _" 
        $ = "new XMLHttpRequest();" 
        $ = "'\\x2e\\x7a\\x69\\x70'" 
        $ = "'\\x4f\\x70\\x65\\x6e'" 
        $ = "\\x43\\x3a\\x5c\\x57" 

    condition:  
        All of them 
} 
```

```text
rule M_APT_Downloader_WINELOADER_1 
{ 
    meta: 
        author = "Mandiant" 
        disclaimer = "This rule is meant for hunting and is not tested to run in a production environment." 
        description = "Detects rc4 decryption logic in WINELOADER samples" 
    
    strings: 
        $ = {B9 00 01 00 00 99 F7 F9 8B 44 24 [50-200] 0F B6 00 3D FF 00 00 00} // Key initialization 
        $ = {0F B6 00 3D FF 00 00 00} // Key size 

    condition: 
        All of them 
} 
```

```text
rule M_APT_Downloader_WINELOADER_2 
{ 
    meta: 
        author = "Mandiant" 
        disclaimer = "This rule is meant for hunting and is not tested to run in a production environment." 
        description = "Detects payload invocation stub in WINELOADER" 

    strings: 
        // 48 8D 0D ?? ?? 00 00  lea rcx, module_start (Pointer to encrypted resource) 
        // 48 C7 C2 ?? ?? 00 00  mov rdx, ???? (size of encrypted source) 
        // E8 [4]  call decryption 
        // 48 8D 05 [4]  lea rcx, ?? 
        // 48 8D 0D [4]  lea rax, module_start (decrypted resource) 
        // 48 89 05 [4]  mov ptr_mod, rax 
        // 
        $ = {48 8D 0D ?? ?? 00 00 48 C7 C2 ?? ?? 00 00 E8 [4] 48 8d 0D [4] 48 8D 05 [4] 48 89 05 } 

    condition: 
        All of them 
}
```

### Responda las siguientes preguntas
> ¿Qué técnica describe el ID T1134?
> **Access Token Manipulation**

> ¿Qué M_APT_Dropper_Rootsaw_Obfuscated detectar la regla de detección?
> **Detects obfuscated ROOTSAW payloads**


# Oportunidades para la búsqueda de amenazas
En función de la inteligencia de amenazas proporcionada en la Tarea 2, buscaremos oportunidades para montar una búsqueda de amenazas. Al principio, la cantidad de inteligencia suministrada puede parecer desalentadora de procesar. `¿Por dónde empezamos?`, `¿Cómo empezamos?`, `¿Qué información necesitamos?`, son algunas preguntas que deben responderse primero. Antes de que se puedan responder estas preguntas para este escenario, se requiere una pequeña descripción general de los estilos y procesos de búsqueda de amenazas.

## Estilos de búsqueda de amenazas

<center>
  <img sr="/assets/images/Yara/Estilos.png">
</center>

Hay tres estilos de caza de amenazas: *caza estructurada*, *caza no estructurada* y *caza basada en situaciones/entidades*.

- `Caza estructurada`: Este estilo de caza utiliza **indicadores de ataque** y **TTP (tácticas, técnicas y procedimientos)** para *buscar posibles ataques* de los actores de amenazas. A esto también se le llama ***caza basada en hipótesis***. La ventaja de este estilo de caza es que un ataque se puede detectar temprano en la cadena de muerte *(Kill Chain)*, evitando daños. Una de las principales fuentes de inteligencia de amenazas utilizadas para este estilo es el marco [MITRE ATT&CK](https://attack.mitre.org/).

- `Caza no estructurada `: Este estilo de caza utiliza **indicadores de compromiso** para impulsar una búsqueda en el entorno. Esto se traduce en varias actividades de búsqueda en toda la infraestructura: uso de reglas YARA para la *coincidencia de patrones*, *escritura de consultas específicas* para aplicar a los datos agregados en el SIEM, etc. Otro nombre para este estilo de caza es la ***caza de amenazas basada en inteligencia***.

```text
SIEM:

      Sistema de gestión de eventos e información de seguridad que se utiliza para agregar información de seguridad en forma de registros, alertas, artefactos y eventos
  en una plataforma centralizada que permitiría a los analistas de seguridad realizar análisis casi en tiempo real durante el monitoreo de seguridad.
```

Las fuentes de inteligencia de amenazas utilizadas para este estilo son los *blogs de seguridad*, la plataforma de intercambio de inteligencia de malware (MISP) y fuentes de inteligencia de amenazas como [abuse.ch](https://abuse.ch/) o [Alienvault](https://otx.alienvault.com/).

- `Búsqueda situacional o basada en entidades `: Este estilo de caza combina varios elementos de la caza estructurada y no estructurada y está impulsada por los **cambios en el panorama de amenazas**. Por ejemplo, un nuevo actor de amenazas, un nuevo informe sobre una amenaza dirigida a su vertical de negocio, información del CSIRT nacional, una solicitud de un cliente y más.

Las actividades incluyen la **formulación de una hipótesis** que detalle qué actores de amenazas podrían dirigirse a su infraestructura y qué activos de alto valor se dirigen, la búsqueda de IOC y la creación o el uso de un perfil de amenazas con la ayuda del marco [MITRE ATT&CK](https://attack.mitre.org/). Las actividades de caza a menudo se ***centran en las Joyas de la Corona*** (los activos más críticos).

```text
MITRE:

      Tácticas, técnicas y conocimiento común de los adversarios de MITRE (ATT&CK)
```

Las principales fuentes de inteligencia de amenazas son los informes de amenazas dentro de la misma vertical de negocio y los ataques históricos.

## Proceso de búsqueda de amenazas

<center>
  <img sr="/assets/images/Yara/Busqueda.png">
</center>

La caza de amenazas consta de 3 fases:

- `Disparador`: Esto es lo que inicia la búsqueda de amenazas. Puede ser un **IOC**, un conjunto de TTPs, una hipótesis, un sistema que se comporta de forma anómala, artículos en blogs externos, informes de terceros, etc.

```text
IOC:

    El indicador de compromiso es un artefacto forense observado en una red o en un sistema operativo que, con alta confianza, indica una intrusión informática.
```

- `Investigación`: Se selecciona un desencadenante específico y se utiliza como punto de partida para las actividades de caza. El cazador de amenazas puede utilizar varias herramientas para respaldar la búsqueda de anomalías, como las reglas YARA, la volatilidad, los escáneres de malware, los analizadores de paquetes como Wireshark y muchos más.

- `Resolución`: Si el cazador de amenazas encuentra pruebas de una infracción, se notifica al equipo de respuesta a incidentes y se inicia el procedimiento de respuesta a incidentes. Dependiendo del procedimiento, el cazador de amenazas puede apoyar al equipo de **IR** mediante el alcance y la profundización de las pruebas encontradas.

```text
IR:

      La respuesta a incidentes (IR) es un enfoque estructurado para gestionar y abordar las secuelas de una violación de seguridad o un ciberataque, también conocido 
  como incidente informático, incidente informático o incidente de seguridad. Implica identificar, investigar, manejar y aprender de eventos o incidentes de seguridad 
  para evitar que ocurra algo similar.
```

## Oportunidades
Apliquemos ahora los conceptos anteriores a nuestro escenario. Hay varias oportunidades proporcionadas dentro de la inteligencia de amenazas recibida:

1. La inteligencia de amenazas recibida detalla TTP específicos atribuidos a APT29, que se sabe que se dirige a entidades políticas.
  - Esta inteligencia permite un *estilo de búsqueda estructurado* utilizando los TTP incluidos en el informe para construir una hipótesis.

2. La información de amenazas recibida incluye indicadores de compromiso y reglas YARA para buscar malware.
  - Esta inteligencia permite un *estilo de búsqueda no estructurado* utilizando los IOC proporcionados.

3. Las dos oportunidades anteriores se pueden combinar para permitir un estilo de caza situacional o basado en entidades.

A lo largo del resto de esta sala, nos centraremos en la ***oportunidad número 2***. Los indicadores de compromiso proporcionados permiten múltiples actividades de búsqueda de amenazas, por ejemplo, ingerirlos en el **IDS**, escanear manualmente con YARA o SIGMA, crear reglas SNORT y más.

```text
IDS:

      El sistema de detección de intrusos (IDS) es un sistema que detecta intrusiones no autorizadas en la red y el sistema. Por ejemplo, la detección de dispositivos 
  no autorizados conectados a la red local y usuarios no autorizados que acceden a un sistema o modifican un archivo.
```

Usaremos las reglas YARA proporcionadas para esta sala para buscar el malware WINELOADER.

### Responda las siguientes preguntas
> ¿Qué estilo de búsqueda de amenazas es proactivo y utiliza indicadores de ataque y TTP?
> **structured hunting**

> ¿En qué fase del proceso de threat hunting se utilizan herramientas como YARA o Volatility?
> **Investigation**

> Ha recibido un informe de inteligencia de amenazas que consta únicamente de indicadores de compromiso. ¿Qué estilo de threat hunting recomienda utilizar?
> **unstructured hunting**

# YARA: Introducción
La inteligencia de amenazas recibida en la Tarea 2 contenía tres reglas YARA. Estas reglas YARA se pueden usar para buscar malware específico (en este escenario, el malware es **WINELOADER**). Antes de ponernos manos a la obra con estas reglas de YARA, es importante entender ***qué es YARA***.

`YARA (Yet Another Ridiculous Acrónimo)`. Es una herramienta que ***Víctor Álvarez*** de *VirusTotal* desarrolló para ayudar a los investigadores de malware a detectar y describir familias de malware.

La funcionalidad principal de YARA se basa en la **coincidencia avanzada de patrones**, explícitamente adaptada al malware. Se puede comparar mejor con el uso de un *grep sobrealimentado* con expresiones regulares complejas en Linux. Al igual que el comando **grep**, el binario YARA iterará sobre todos los archivos en una ruta designada, tratando de encontrar una coincidencia con la información proporcionada en la *regla YARA*.

Una regla YARA describe una familia de malware basada en un patrón que utiliza un conjunto de cadenas y lógica booleana.

## Estructura de una regla YARA
Una regla YARA utiliza lenguaje descriptivo para definir un patrón que consta de cadenas para que coincida con una condición booleana especificada al final de la regla. Las partes principales de una regla YARA son el `nombre de la regla`, el `meta`, las `cadenas` y la `condición`. A continuación, discutiremos cada parte.

- `Nombre de la regla`: El nombre de la regla es un *nombre descriptivo de la regla* y comienza con la palabra clave regla. Las prácticas recomendadas incluyen establecer un nombre que aclare para qué se usa la regla.

- `Meta`: Esta parte define información adicional como *descripción*, *autor* y *más*. Los identificadores personalizados y los pares de valores se pueden crear libremente. La información definida en meta **no se puede utilizar en la parte de condición**. Si incluir esta parte o no depende completamente de usted. La regla funcionará completamente bien sin él. Sin embargo, se recomienda incluir la parte meta con información básica, incluido el autor y la descripción de para qué usar la regla.

- `Instrumentos de cuerda`: En esta parte de la regla, se definen las *cadenas coincidentes*. Se pueden definir varios tipos de cadenas, lo cual es esencial para crear reglas funcionales.

- `Condición`: En esta parte de la regla, se define una condición coincidente mediante los identificadores definidos en la parte de cadenas.

## Ejemplo de una regla YARA
A continuación se muestra un ejemplo de una regla YARA que recibimos del equipo de **CTI**. En este ejemplo, están presentes las cuatro partes discutidas en el párrafo anterior:

```text
CTI:

      La inteligencia de amenazas cibernéticas es el conocimiento basado en evidencia sobre los adversarios, incluidos sus indicadores, tácticas, motivaciones y 
  consejos prácticos contra ellos.
```

- `Nombre de la regla`: **M_APT_Dropper_Rootsaw_Obfuscated**. El título de la regla está bien elegido y le da al usuario una buena idea de para qué usarlo. En este caso, es para detectar un gotero llamado *Rootsaw* que está ofuscado.

- `Meta`: Es una buena práctica incluir datos relevantes que proporcionen más información sobre la regla. Esto ayuda al usuario de la regla YARA a saber *para qué usar la regla*, *quién la escribió* y *dónde aplicarla*.

- `Cadenas`: las cadenas incluidas en este ejemplo ayudan al usuario a encontrar un archivo que contenga esas cadenas. *¿Cómo eligen los analistas de malware esas cadenas?* Analizan el malware y determinan qué lo identifica de forma única. Las cadenas utilizadas son cadenas de texto. Las dos primeras líneas son sencillas.

- `Condición`: Esta regla requiere que todas las cadenas definidas estén presentes para tener una coincidencia. Esto significa que todas las cadenas definidas en la parte 3 *deben tener una coincidencia en el mismo archivo con el que se coincide*.

<center>
  <img sr="/assets/images/Yara/Ejemplo.png">
</center>

Solo se requieren dos partes para que una regla funcione: `nombre de la regla` y la `condición`. Todas las demás partes *son opcionales*. Sin embargo, se recomienda agregar cadenas a una regla si desea crear reglas YARA complejas y funcionales.

Puedes encontrar información más detallada sobre cómo escribir las reglas de YARA en la documentación oficial de [YARA](https://yara.readthedocs.io/en/stable/writingrules.html), o si prefieres no escribirlas tú mismo, hay muchos repositorios disponibles con reglas de YARA bien escritas. **Florian Roth** es una buena autoridad en la redacción de las reglas de calidad de YARA. Florian también creó una herramienta llamada ***YARA FORGE*** que agiliza la recopilación pública de reglas YARA. Esta herramienta recopila, prueba, organiza y redistribuye estas reglas de manera más eficiente, haciéndolas más accesibles y valiosas para la comunidad de seguridad cibernética. Puedes encontrar documentación relacionada con esta herramienta dentro del perfil oficial de GitHub de [YARAHQ](https://github.com/YARAHQ/yara-forge).

### Responda las siguientes preguntas
> Aparte del nombre de la regla, ¿qué otra sección también es necesaria en una regla YARA?
> **condition**

# YARA: Cuerdas y condiciones
En la tarea anterior, discutimos brevemente las diferentes partes de una regla YARA y mencionamos que una regla YARA requiere que al menos la parte de cadenas y condición funcione según lo previsto. Estas dos partes hacen o deshacen una regla de YARA, por lo que es importante profundizar un poco más en ellas. Pero antes de eso, hablaremos brevemente de los falsos positivos.

- `Falsos positivos`: Una parte esencial de la búsqueda de amenazas es *excluir los falsos positivos*. Cuando nos fijamos en YARA, esto significa **crear reglas que identifiquen de forma única la amenaza que está buscando**. Es más fácil decirlo que hacerlo. Las reglas de YARA pueden volverse complejas muy rápidamente cuando se adaptan a malware específico. El uso de reglas YARA bien escritas y probadas es imprescindible. Esto nos deja con dos opciones: o bien *profundizar en los detalles de la redacción de las reglas de YARA*, o bien *utilizar las reglas de YARA creadas por expertos* (como las incluidas en la inteligencia de amenazas que recibimos en la Tarea 3). Por supuesto, la combinación de estas dos opciones también es viable. Incluso cuando solo usas reglas YARA precreadas, es crucial que las entiendas.

## Instrumentos de cuerda
En este párrafo, investigaremos qué cadenas podemos usar dentro de una regla YARA. Discutiremos los tipos generales de cadenas disponibles sin profundizar demasiado en cada categoría. La mayoría de las palabras clave modificadoras que encontraremos a lo largo de este párrafo se pueden combinar. Está fuera del alcance discutir todas las combinaciones posibles. Si quieres profundizar más, puedes echar un vistazo a la documentación oficial de YARA.

- `Cadenas de texto`: En su forma más simple, podemos definir una *cadena codificada en ASCII* que coincida con algún texto que buscamos. Es esencial mencionar que la cadena especificada distingue entre mayúsculas y minúsculas. Veamos un ejemplo:

```text
rule textString
    {
      strings: 
            $1 = "This is an ASCII-encoded string" //strings are defined between double quotes
            $2 = "This is an ascii-encoded string" //not the same as $1.
            
      condition:  
            all of them 
    }
```

Es posible definir la cadena como que no distingue entre mayúsculas y minúsculas agregando el modificador `nocase` junto a ella. De esta manera, buscará todas las permutaciones de la cadena especificada. En el ejemplo siguiente se muestra el uso del modificador **nocase**:

```text
rule noCaseTextString
    {
      strings: 
            $1 = "This is an ASCII-encoded string" nocase
            
      condition:  
            $1
    }
```

- `Cadenas de caracteres anchos`: Las cadenas se pueden codificar de diferentes maneras. Una forma que se encuentra a menudo en los binarios es codificar la cadena como dos bytes por carácter en lugar de la codificación ASCII tradicional de un byte. Por ejemplo, la cadena `tryhackme` se codificaría como `t00r00y00h00a00c00k00m00e00`. Es posible utilizar un modificador junto a la cadena definida para que la regla coincida con esta cadena de caracteres anchos. El modificador utilizado para esto es `wide`. En el ejemplo siguiente se muestra el uso del modificador **wide**:

```text
rule wideTextString
    {
      strings: 
            $1 = "tryhackme" wide // will match with t\x00r\x00y\x00h\x00a\x00c\x00k\x00m\x00e\x00 
            
      condition:  
            $1
    } 
```

- `Cadenas hexadecimales`: Cuando los analistas de malware comienzan a analizar malware, a menudo usan un desensamblador y depurador como IDA Pro para desmantelar los archivos binarios. A menudo, los fragmentos de código que descubren se muestran en hexadecimal. A continuación, podemos utilizar las cadenas hexadecimales descubiertas durante el análisis para crear nuestras propias reglas YARA. Las secuencias de caracteres hexadecimales suelen ser más difíciles de ofuscar y ocultar para los atacantes. Por lo tanto, estas cadenas hexadecimales proporcionan una excelente oportunidad para identificar de forma única un determinado binario malintencionado. Veamos un ejemplo de cadenas hexadecimales en una regla YARA.

```text
rule hexString
    {
      strings: 
            $1 = { E2 34 B6 C8 A3 FB } // Hexadecimal strings are defined between {}
            
      condition:  
            $1
    } 
```

La definición de cadenas hexadecimales puede ser muy flexible. YARA admite cuatro formas de realizar esto: Uso de comodines, no operadores, saltos y alternativas. Todas las construcciones también se pueden combinar. Veamos un ejemplo de su uso principal:

```text
rule hexStringExpanded
    {
      strings: 
            $1 = { E2 34 B6 ?? A3 FB } // The ? is a wildcard and can represent any hex value.
            $2 = { E2 34 B6 ~00 A3 FB } // The ~ is a not operator that precedes the value to exclude from the search. In this case 00.
            $3 = { E2 34 [2-4] A3 FB } // The [X-Y] construct defines a jump. This means that any value between 2 and 4 bytes can occupy this position.
            $4 = { E2 34 (C5|B5) A3 FB } // Between () alternative byte sequences can be defined separated with the boolean operator OR. The value can be B5 OR C5.       
      condition:   
             $1
    } 
```

- `XOR Instrumentos de cuerda`: Los creadores de malware a menudo usan XOR para cifrar su código, lo que dificulta el análisis de los analistas de malware. También ayuda a evadir las firmas del antivirus. El soporte de cadenas XOR en YARA nos ayuda a buscar variaciones de cadenas cifradas XOR con claves de 1 byte. Veamos un ejemplo:

```text
XOR:

      Es una operación binaria que se usa comúnmente para el cifrado y descifrado de datos. XOR opera con datos binarios (bits) y se basa en los principios del 
  álgebra booleana. La operación implica dos bits. El resultado de la operación es "1" si los dos bits son diferentes y "0" si son iguales.
```

```text
rule xorString
    {
      strings: 
            $1 = "http://maliciousurl.thm" xor // This line will look for all variations possible with a 1-byte XOR key
            
      condition:  
            $1
    } 
```

Los autores de malware suelen utilizar la codificación para evadir la detección. Una técnica de codificación que se utiliza a menudo es base64. YARA admite la búsqueda de cadenas codificadas en base64. Para hacer esto, puede usar el modificador base64 después de definir la cadena. YARA buscará la cadena codificada en base64 cuando se ejecute la regla YARA. Veamos un ejemplo:

```text
rule base64String
        {
            strings: 
                $1 = "This is a regular string"  base64 // At runtime YARA will encode the string with base64 and look for matches.
                
            condition:  
                $1
        } 
```

- `Expresiones regulares`: Al igual que con el comando grep en Linux, el uso de expresiones regulares hace que YARA sea poderoso. Puede definir expresiones regulares de la misma manera que cadenas, con la única diferencia de barras diagonales en lugar de comillas dobles. Una ventaja es que los modificadores anteriores también se pueden aplicar a estas expresiones regulares. Echa un vistazo a nuestra sala de [expresiones regulares](https://tryhackme.com/r/room/catregex) para obtener más información sobre cómo crear expresiones regulares. Sin embargo, es importante tener en cuenta que desde la versión 2.0, YARA ha utilizado su motor de expresiones regulares, que implementa la mayoría de las características que se encuentran en PCRE. Por ahora, veamos un ejemplo de una regla YARA que incluye una expresión regular:

```text
rule regularExpression
        {
            strings: 
                $1 = /THM\{[a-zA-Z]{3}\}/ // This regex will match any string that starts with "THM{", ends with "}" and has 3 alphabetic characters (lower-case or upper-case) between the curly brackets.
                
            condition:  
                $1
        } 
```

## Condiciones
Una vez que haya definido sus cadenas, es crucial definir cómo combinarlas y hacer coincidir los archivos que está buscando. YARA ofrece una gran flexibilidad a la hora de realizar diferentes combinaciones. YARA incluye operadores booleanos, relacionales, aritméticos y bit a bit. Además, se pueden utilizar algunas palabras clave. La siguiente tabla muestra una descripción general de los operadores y las palabras clave:

|  Operadores Booleanos | Operadores Relacionales | Operadores Aritméticos | Oberadores bit a bit | Palabras clave      |
|-----------------------|-------------------------|------------------------|----------------------|---------------------|
|           y           |           >=            |            +           |          &           | 1 de ellos          |
|           o           |           <=            |            -           |          |           | Cualquiera de ellos |
|          no           |            <            |            *           |          <<          | Ninguno de ellos    |
|                       |            >            |            \           |          >>          | Contiene            |
|                       |           ==            |            %           |          ~           | icontiene           |
|                       |           !=            |                        |          ^           | Comienza con        |
|                       |                         |                        |                      | istartscon          |
|                       |                         |                        |                      | Termina con         |
|                       |                         |                        |                      | iendswith           |
|                       |                         |                        |                      | ies iguales         |
|                       |                         |                        |                      | Partidos            |
|                       |                         |                        |                      | No definido         |
|                       |                         |                        |                      | Tamaño de archivo   |

Podríamos dedicar una sala completa a todos los operadores, pero para esta sala, nos centraremos solo en algunos de los operadores booleanos y palabras clave. Veamos algunos ejemplos:

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            all of them // Matches when all defined strings are present.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            any of them  // Matches when at least one of the defined strings is present.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
             1 of $(*) // Identical to "any of them" condition.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            "$1 or $2" // Matches when 'Try' or 'Hack' is present.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            $1 and $2 // Matches when 'Try' and 'Hack' are present.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            $1 and ($2 or $3) // Matches when 'Try' and 'Hack' or 'Try' and 'Me' combinations are present.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            none of them // Matches only when none of the defined strings are present.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            filesize < 500KB // Matches all files smaller than 500 KiloByte. This can only be used when matching for files.
    } 
```

```text
rule differentConditions
    {
      strings: 
            $1 = "Try" 
            $2 = "Hack" 
            $3 = "Me" 
            
      condition:  
            ($1 or $2) and filesize < 200KB // Matches for 'Try' or 'Hack' in files smaller than 200KB.
    } 
```

Ahora que hemos cubierto las dos partes más importantes de una regla YARA, pasemos a la siguiente tarea y examinemos cómo podemos usar las reglas YARA para buscar Indicadores de Compromiso.

### Responda las siguientes preguntas
> ¿Qué modificador se debe usar si desea buscar caracteres codificados de 2 bytes?
> **wide**

> ¿Qué condición se debe utilizar si desea excluir las cadenas definidas del proceso de coincidencia?
> **none of them**

# Entorno y configuración
En esta sala, utilizaremos una máquina virtual Windows Server 2019 con YARA instalado. Esta máquina virtual se utilizará para todas las tareas y ejercicios enumerados. Puede iniciar la máquina haciendo clic en el botón de abajo `Start Machine`. La máquina virtual tardará aproximadamente 2 minutos en iniciarse y se iniciará en vista dividida. Si la máquina virtual no está visible, use el botón azul en la parte superior de la página `Show Split View`.

<center>
  <img sr="/assets/images/Yara/Start.jpeg">
</center>

Como alternativa, puede conectarse a la máquina virtual a través de Escritorio remoto (RDP) con las siguientes credenciales:

```text
RDP:

    El Protocolo de escritorio remoto es un protocolo utilizado para establecer sesiones gráficas remotas a través de la red.
```

<center>
  <img sr="/assets/images/Yara/RDP-2.jpeg">
</center>

Todos los archivos necesarios utilizados en esta sala se encuentran en `C:\TMP\`.

## Instalar YARA localmente
Si desea explorar más a fondo YARA después de terminar esta habitación, puede instalarlo en su propio sistema. YARA se puede instalar en Windows, distribuciones de Linux y MAC OS. Cada instalación es ligeramente diferente. Puedes encontrar las instrucciones para instalar YARA en la [página oficial](https://yara.readthedocs.io/en/latest/gettingstarted.html) de documentación de YARA.

YARA también se puede utilizar desde un script de Python. Para que esto sea posible, instale la extensión yara-python, que está disponible en el perfil de [GitHub de VirusTotal](https://github.com/VirusTotal/yara-python).

### Responda las siguientes preguntas
> Inicie la máquina virtual y continúe con la siguiente tarea.