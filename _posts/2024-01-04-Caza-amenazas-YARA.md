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

![Portada](https://tryhackme.4kiing.net/assets/images/Yara/Portada.jpeg)

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

Para seguir con la lectura del artículo completo visita el siguiente [sitio](https://y4r4.4kiing.monster/).