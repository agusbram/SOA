# Grupo 1 — Telegraf + InfluxDB 2
## Sistema de recolección y almacenamiento de métricas

> **Materia:** Arquitectura Orientada a Servicios   
> **Tecnologías:** Telegraf 1.38 · InfluxDB 2.8 · Docker Compose    
> **Rol en el TP integrado:** Recolección de métricas del host y almacenamiento en series temporales para consumo del Grupo 2 (Grafana) 

---

## ¿Qué hace este proyecto?

Este repositorio implementa la capa de **recolección y almacenamiento** del sistema de observabilidad del TP integrado. El flujo es el siguiente:

```
[Sistema Operativo del Host]
         │
         │  Telegraf lee /proc y /sys cada 10 segundos
         ▼
    [ Telegraf ]
         │
         │  Escribe via HTTP usando InfluxDB Line Protocol
         ▼
   [ InfluxDB 2.x ]
         │
         │  El Grupo 2 (Grafana) se conecta desde acá
         ▼
    [ Grafana ]
```

Telegraf recolecta métricas de CPU, memoria, disco, red, carga del sistema y procesos, y las almacena como series temporales (una secuencia de valores medidos a lo largo del tiempo) en InfluxDB, organizadas por host.

---

## Requisitos y dependencias

El único requisito para correr este proyecto es **Docker con Docker Compose**. No hace falta instalar Telegraf ni InfluxDB directamente en el sistema.

### Verificar instalación

```bash
docker --version
docker compose version
```

---

## Estructura del proyecto

```
SOA/
├── docker-compose.yml        # Orquesta los servicios (InfluxDB + Telegraf)
├── .env                      # Variables reales
├── .env.example              # Plantilla sin valores
├── .gitignore                # Excluye .env del repositorio
├── telegraf/
│   └── telegraf.conf         # Configuración del agente recolector
└── README.md                 # Este archivo
```

### Rol de cada archivo

**`docker-compose.yml`** — define la infraestructura completa: qué imágenes usar, cómo se conectan los servicios, qué puertos exponer, cómo persistir los datos y cuándo está listo cada servicio.

**`.env`** — contiene las credenciales reales (contraseña, token, nombres). Se lo crea localmente copiando `.env.example`.

**`.env.example`** — plantilla vacía que documenta qué variables existen y con qué formato. Es la referencia para cualquiera que clone el repositorio.

**`.gitignore`** — garantiza que `.env` nunca sea subido accidentalmente al repositorio.

**`telegraf/telegraf.conf`** — define qué métricas recolectar (inputs) y a dónde enviarlas (output). Es el corazón del sistema de recolección.

---

## Instalación desde cero

### Paso 1: clonar el repositorio

```bash
git clone https://github.com/agusbram/SOA
cd SOA
```

### Paso 2: crear el archivo de variables de entorno

```bash
cp .env.example .env
```

Abrir `.env` y completar los valores:

```bash
nano .env
```

Generar un token seguro con:

```bash
openssl rand -hex 32
```

Copiar el resultado y pegarlo como valor de `INFLUXDB_TOKEN` en el `.env`.

### Paso 3: levantar los servicios

```bash
docker compose up -d
```

Docker va a descargar las imágenes la primera vez (puede tardar unos minutos según la conexión) y luego levantará los dos contenedores en segundo plano.

### Paso 4: verificar que todo esté funcionando

```bash
# Ver el estado de los contenedores
docker compose ps

# El resultado esperado es:
# influxdb   running (healthy)
# telegraf   running
```

InfluxDB puede tardar hasta 40 segundos en pasar a estado `healthy` la primera vez. Telegraf no arranca hasta que InfluxDB esté listo (por el `depends_on` con `condition: service_healthy`).

### Paso 5: verificar que los datos lleguen

Abrir el navegador en `http://localhost:8086` e ingresar con las credenciales del `.env`. Ir a **Data Explorer**, seleccionar el bucket `metricas_hosts` y verificar que aparecen las measurements: `cpu`, `disk`, `diskio`, `mem`, `net`, `processes`, `system`.

---

## Variables de entorno impuestas por la tecnología

| Variable | Valor | Motivo |
|----------|-------|--------|
| `INFLUXDB_PORT` | `8086` | Puerto estándar de InfluxDB, definido por la herramienta |
| `INFLUXDB_TOKEN` | generado con `openssl` | Debe ser un valor aleatorio seguro |

---

## Puertos expuestos

| Puerto | Servicio | Uso |
|--------|----------|-----|
| `8086` | InfluxDB | API HTTP e interfaz web |

Telegraf no expone ningún puerto porque solo escribe datos hacia InfluxDB, no recibe conexiones externas.

---

## Fuentes y referencias

- Documentación oficial de Telegraf: https://docs.influxdata.com/telegraf/v1/
- Plugins de Telegraf: https://docs.influxdata.com/telegraf/v1/plugins/
- Documentación oficial de InfluxDB 2.x: https://docs.influxdata.com/influxdb/v2/
- Conceptos clave de InfluxDB: https://docs.influxdata.com/influxdb/v2/reference/key-concepts/
- Imagen Docker de InfluxDB: https://hub.docker.com/_/influxdb
- Imagen Docker de Telegraf: https://hub.docker.com/_/telegraf
- Docker Compose — healthcheck: https://docs.docker.com/compose/compose-file/05-services/#healthcheck
- Docker Compose — networking: https://docs.docker.com/compose/networking/
- Docker Compose — volumes: https://docs.docker.com/storage/volumes/
- InfluxDB Line Protocol: https://docs.influxdata.com/influxdb/v2/reference/syntax/line-protocol/
- CLI influx ping: https://docs.influxdata.com/influxdb/v2/reference/cli/influx/ping/

### Referencias adicionales

**InfluxDB, Telegraf y Grafana — integración:**
- El dashboard perfecto (instalación sobre Ubuntu): https://www.jorgedelacruz.es/2020/11/23/en-busca-del-dashboard-perfecto-influxdb-telegraf-y-grafana-parte-i-instalando-influxdb-telegraf-y-grafana-sobre-ubuntu-20-04-lts/
- Monitorización de datos con InfluxDB, Telegraf y Grafana: https://openwebinars.net/blog/monitorizando-datos-con-influxdb-telegraf-y-grafana/
- Telegraf Metrics Dashboard for InfluxDB 2.0 (Flux) — Grafana Dashboards: https://grafana.com/grafana/dashboards/15650-telegraf-influxdb-2-0-flux/

**InfluxDB:**
- Referencia de opciones de configuración (http-bind-address): https://docs.influxdata.com/influxdb/v2/reference/config-options/#http-bind-address
- Instalación con Docker: https://docs.influxdata.com/influxdb/v2/install/?t=Docker

**Telegraf:**
- Estrategias de despliegue con Docker Compose: https://www.influxdata.com/blog/telegraf-deployment-strategies-docker-compose/
- Instalación con Docker: https://docs.influxdata.com/telegraf/v1/install/?t=Docker
- Ejemplo de docker-compose con Telegraf, InfluxDB y Grafana: https://github.com/fcuiller/docker-compose-telegraf-influxdb-grafana/blob/master/docker-compose.yml
- Tutorial en video — Telegraf + InfluxDB: https://www.youtube.com/watch?v=xaHu52DA1pg
- Referencia de configuración de Telegraf: https://docs.influxdata.com/telegraf/v1/configuration/
- Docker monitoring tutorial con Telegraf e InfluxDB (CNCF): https://www.cncf.io/blog/2022/06/10/docker-monitoring-tutorial-how-to-monitor-docker-with-telegraf-and-influxdb/
