# Inventario de servicios - AgriSmart Latacunga

Equipo 1 - Infraestructura y servicios cloud (H2)

Lista de los servicios que usa AgriSmart, separados por nivel. Sale del documento del proyecto (versión 3), sección 6 y los ADR. A cada servicio le puse un ID para que los otros equipos lo puedan citar.

| ID | Servicio | Nivel | Para qué sirve | Dónde corre | Referencia |
|---|---|---|---|---|---|
| S01 | Nodos ESP32 y sensores (pH, humedad, temperatura, NPK), 40 fijos y 4 kits portátiles | Edge | Miden el suelo cada 15 min y guardan los datos si se corta la red | Parcela | Sección 6 |
| S02 | Gateway LoRaWAN con ChirpStack | Fog | Recibe los datos que mandan los nodos | Servidores HP | ADR-003 |
| S03 | Broker MQTT | Fog | Pasa los eventos de un servicio a otro | Servidores HP | ADR-004 |
| S04 | Ingesta, alertas y recomendaciones | Fog | Procesa las lecturas y avisa aunque no haya internet | Servidores HP | Sección 6, ADR-008 |
| S05 | TimescaleDB | Fog | Guarda las lecturas en series de tiempo | Servidores HP | ADR-005 |
| S06 | Grafana | Fog | Muestra los datos en paneles | Servidores HP | Sección 6, ADR-011 |
| S07 | Sincronizador | Fog | Manda los datos a la nube cuando hay internet | Servidores HP | Sección 6, ADR-006 |
| S08 | Docker Compose | Fog | Levanta todos los servicios del fog | Servidores HP | ADR-012 |
| S09 | API Gateway y BFF móvil | Cloud | Es la entrada de la app y del panel web | AWS | Sección 6 |
| S10 | PostgreSQL (RDS) | Cloud | Guarda los datos completos del estudio | AWS | Sección 6, ADR-005 |
| S11 | S3 | Cloud | Guarda los modelos de IA y los datasets | AWS | Sección 6, ADR-008 |
| S12 | Cognito | Cloud | Login de los usuarios | AWS | Sección 6, ADR-010 |
| S13 | Secrets Manager | Cloud | Guarda las claves y contraseñas | AWS | ADR-010 |
| S14 | Entrenamiento y registro de modelos | Cloud | Entrena y guarda las versiones de la IA | AWS | Sección 6, ADR-008 |
| S15 | GitHub Actions | Transversal | Pruebas y despliegue automático | GitHub | ADR-012 |
| S16 | OpenTelemetry y Prometheus | Transversal | Logs, métricas y trazas | Por definir | ADR-011 |
| S17 | App Flutter con SQLite | Cliente | La app funciona sin conexión en el celular | Teléfono | Sección 6, ADR-007 |
| S18 | Panel web de investigadores | Cliente | Consultar y exportar los datos | Navegador | Sección 6 |
| S19 | Firebase | Cliente | Solo para las notificaciones | Google | ADR-007 |

## Pendiente

- Falta saber dónde corre el BFF/API en AWS (lo decide el Equipo 4). El documento dice "servicios administrados en AWS" en el ADR-012, pero en el resumen habla de backend con Docker en AWS.
- De S09 a S14 falta confirmar si son contenedores o servicios administrados.
- El documento no dice dónde corre S16 (fog, AWS o los dos).
- Falta confirmar dónde se instala el nodo fog (UTC o centro de la parroquia) y su energía de respaldo.

La clasificación IaaS/PaaS/SaaS la hace el Equipo 2 y quién administra cada servicio el Equipo 3, por eso no están aquí.
