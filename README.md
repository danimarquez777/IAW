# IAW

# Práctica 1: Servidores, Proxies y Certificados (NimbusDocs)

## I. Diseño y Requisitos

### 1. Dominios elegidos
Se han seleccionado dos dominios ficticios representativos de las dos marcas de la organización:
* **Marca Pública:** `nimbus-tech.local`
  * **Justificación:** Dominio destinado a la documentación técnica general orientada al público y usuarios externos.
* **Marca Interna:** `nimbus-legal.local`
  * **Justificación:** Dominio reservado para la documentación legal e interna de la empresa.

---

### 2. Elección y justificación del MPM de Apache
* **MPM Seleccionado:** `mpm_event`
* **Justificación técnica:** 
  * `mpm_event` es la evolución del modelo de hilos (`mpm_worker`), optimizado para gestionar eficientemente conexiones en estado *Keep-Alive*.
  * Es la opción estándar más eficiente cuando Apache se coloca como servidor web de backend detrás de un proxy inverso como Nginx.

---

## II. Fase 1 - Backend Apache Multi-Marca

### Estructura de Virtual Hosts
* **`nimbus-tech.local`**: Configurado con `DocumentRoot` en `/var/www/nimbus-tech`.
* **`nimbus-legal.local`**: Configurado con `DocumentRoot` en `/var/www/nimbus-legal`.
  * **Protección de recursos:** Se ha desactivado el listado explícito de directorios y se ha restringido el acceso web directo al fichero de configuración sensible `config-secreto.json`.

### Comportamiento por defecto
* **Archivo de configuración:** `000-default-custom.conf`
* **Explicación técnica:** Apache evalúa las peticiones entrantes comparando la cabecera `Host` con el `ServerName` de los Virtual Hosts activos. Si la petición utiliza una IP o un dominio no registrado, cae en el primer Virtual Host cargado alfabéticamente, asegurando así un comportamiento controlado que no expone accidentalmente ninguna de las dos marcas.