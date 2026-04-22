# Monitoreo de Rendimiento con Grafana y Prometheus en Linux

Guía para monitorear el consumo de recursos de un host Linux mientras ejecutas tu juego desarrollado en React + Three.js. El stack está pensado para correr con Docker Compose y usa Node Exporter para exponer métricas del sistema, Prometheus para recolectarlas y Grafana para visualizarlas.

## Arquitectura general

```text
Juego React/Three.js en el navegador
        │
        │ consume recursos del host Linux
        ▼
┌──────────────────────────────────┐
│ Node Exporter (:9100)           │──► CPU, RAM, disco, red
└──────────────────────────────────┘
        │
        │ scrape cada 5s
        ▼
   Prometheus (:9090)
        │
        │ consultas PromQL
        ▼
   Grafana (:3000) → dashboards + Explore
```

> Scrape significa extraer información de forma automática usando código, en lugar de hacerlo manualmente.

## Qué incluye

- `prometheus.yml` con la configuración de scrape
- `docker-compose.yml` para levantar Prometheus, Grafana y Node Exporter
- Provisioning automático de la fuente de datos en Grafana

## Requisitos previos

- Linux con Docker y Docker Compose
- Navegador web moderno
- Node.js y npm para ejecutar el juego
- GPU NVIDIA opcional si después quieres métricas de GPU

## Arranque rápido

1. Levanta el stack:

```bash
docker compose up -d
```

2. Abre Prometheus en:

http://localhost:9090

3. Abre Grafana en:

http://localhost:3000

Credenciales por defecto:

| Campo | Valor |
| --- | --- |
| Usuario | admin |
| Contraseña | admin |

4. Ejecuta el juego con `npm run dev` o `npm start`.

## Cómo funciona

Node Exporter corre como contenedor y expone métricas del host Linux. Prometheus las scrapea cada 5 segundos y Grafana las consume como fuente de datos. Desde ahí puedes usar Explore o construir dashboards propios.

## Prometheus

Prometheus es el motor que recolecta, almacena y consulta las métricas.

### Configuración

El archivo `prometheus.yml` ya apunta a:

- `prometheus:9090`
- `node-exporter:9100`

### Verificación

Si Prometheus está funcionando, puedes consultar:

```promql
up
```

También es útil probar:

```promql
node_exporter_build_info
```

## Node Exporter

Node Exporter expone métricas del sistema operativo del host Linux.

### Métricas principales

- CPU
- Memoria RAM
- Disco
- Red

### Nota importante

En Linux no se usa Windows Exporter. Si ves referencias a Windows Exporter en algún material anterior, ignóralas para este laboratorio.

## Grafana

Grafana es la herramienta de visualización donde crearás paneles y dashboards.

### Acceso

Abre:

http://localhost:3000

Credenciales por defecto:

| Campo | Valor |
| --- | --- |
| Usuario | admin |
| Contraseña | admin |

### Fuente de datos

La fuente de datos de Prometheus queda configurada automáticamente con provisioning, así que no necesitas crearla a mano.

### Explore

Usa Explore para probar consultas PromQL antes de crear paneles.

## Consultas PromQL útiles

### CPU

```promql
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

### RAM

```promql
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / 1073741824
```

```promql
node_memory_MemTotal_bytes / 1073741824
```

### Disco

```promql
rate(node_disk_read_bytes_total[1m]) / 1048576
```

```promql
rate(node_disk_written_bytes_total[1m]) / 1048576
```

### Red

```promql
rate(node_network_receive_bytes_total{device!~"lo"}[1m]) / 1048576
```

```promql
rate(node_network_transmit_bytes_total{device!~"lo"}[1m]) / 1048576
```

## Cómo usarlo en el laboratorio

Flujo recomendado:

1. Verifica que los contenedores estén corriendo.
2. Abre Grafana y revisa la fuente de datos.
3. Cambia el rango de tiempo a los últimos 15 minutos.
4. Activa auto-refresh cada 5 segundos.
5. Inicia tu juego con `npm run dev` o `npm start`.
6. Interactúa con el juego y observa CPU, RAM, disco y red en tiempo real.

## Puertos

| Servicio | Puerto | URL |
| --- | --- | --- |
| Prometheus | 9090 | http://localhost:9090 |
| Node Exporter | 9100 | http://localhost:9100/metrics |
| Grafana | 3000 | http://localhost:3000 |

## Solución de problemas

Si Prometheus no muestra métricas:

1. Ejecuta `docker compose ps` y confirma que `prometheus` y `node-exporter` estén en estado `Up`.
2. Abre `http://localhost:9090/targets` y verifica que los targets estén en `UP`.
3. Revisa `prometheus.yml` si algún target aparece caído.

Si Grafana no abre:

1. Verifica que el contenedor `grafana` esté en ejecución.
2. Confirma el acceso en `http://localhost:3000`.
3. Usa las credenciales `admin` / `admin`.
