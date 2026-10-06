# Práctica 1 — Fase 1: Backend Apache Multi-Marca
**Asignatura:** Implantación de Aplicaciones Web (IAW)
**Curso:** 2026/2027
**Proyecto:** NimbusDocs Portal Documental Multi-Marca



I. Diseño y Requisitos

### 1. Dominios Ficticios Elegidos y Justificación
Para este despliegue multi-marca bajo la infraestructura de NimbusDocs se han seleccionado dos dominios distintos:
* `docs.local` (Marca Pública): Destinado al portal web de documentación técnica accesible para clientes externoS.
* `vault.local` (Marca Interna): Bóveda documental privada utilizada por los equipos internos de operaciones y devops para la gestión de manuales internos e infraestructuras.

### 2. Elección del MPM de Apache y Justificación Técnica
Se ha seleccionado **`mpm_event`** frente a `prefork` y `worker`.

Justificación técnica:
* Eficiencia de recursos: A diferencia de `prefork` (que asigna un proceso por conexión), `event` utiliza hilos dedicados para atender peticiones y delega el manejo de conexiones Keep-Alive inactivas a un hilo monitorizador separado.
* Preparación para arquitectura con Proxy (Fase 2): Al situar en la Fase 2 un proxy inverso (Nginx) por delante, las conexiones HTTP persistentemente abiertas entre el proxy y el backend con `prefork` consumirían excesivos procesos. `mpm_event` gestiona peticiones concurrente masivas con un consumo de memoria RAM reducido.

---

## II. Fase 1 — Configuración y Despliegue de Virtual Hosts

### 1. Estructura de Directorios y Ficheros de Prueba

Creación de los directorios de contenido:
```bash
sudo mkdir -p /var/www/docs
sudo mkdir -p /var/www/vault
