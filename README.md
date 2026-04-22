# Monitoreo de rendimiento en Linux con Grafana y Prometheus

Este proyecto levanta Prometheus y Grafana con Docker Compose para monitorear recursos de Linux mientras ejecutas tu juego React + Three.js en el navegador. En Linux, el exporter correcto es Node Exporter, no Windows Exporter.

## Arquitectura

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
				│ datasource
				▼
	 Grafana (:3000)
```

## Qué incluye

- `prometheus.yml` para Prometheus
- `docker-compose.yml` para levantar Prometheus, Grafana y Node Exporter
- Provisioning automático de la fuente de datos en Grafana

## Requisitos

- Linux con Docker y Docker Compose
- Navegador web moderno
- Node.js y npm para correr el juego
- GPU NVIDIA opcional si luego quieres métricas de GPU

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

4. Ejecuta tu juego con `npm run dev` o `npm start`.

## Cómo funciona

Node Exporter corre como contenedor y expone métricas del host Linux. Prometheus las scrapea cada 5 segundos y Grafana las consume como fuente de datos.

## Consultas PromQL útiles

### CPU

```promql
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

### RAM

```promql
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
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

## Notas

- No uses `windows_exporter` en Linux.
- Si quieres métricas de GPU NVIDIA en Linux, puedo agregarte un segundo servicio en el stack después.
