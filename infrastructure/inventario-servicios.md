# Inventario de servicios - AgriSmart Latacunga

Equipo 1 - Infraestructura y servicios cloud (H2)



| ID | Servicio | Nivel | Función en AgriSmart | Dónde corre | Referencia |
|---|---|---|---|---|---|
| S01 | Nodos ESP32 + sensores (pH, humedad, temperatura, NPK) | Edge | Medir cada 15 min y reenviar ante cortes | Parcela | Sección 6 |
| S02 | Gateway LoRaWAN + ChirpStack | Fog | Recibir lecturas de los nodos | Servidores HP | ADR-003 |
| S03 | Broker MQTT | Fog | Transportar eventos entre servicios | Servidores HP | ADR-004 |
| S04 | Servicios de ingesta, alertas y recomendaciones | Fog | Procesar lecturas y alertar sin internet | Servidores HP | ADR-008 |
| S05 | TimescaleDB | Fog | Guardar series de tiempo | Servidores HP | ADR-005 |
| S06 | Grafana | Fog | Paneles de monitoreo | Servidores HP | ADR-011 |
| S07 | Sincronizador | Fog | Replicar datos a la nube | Servidores HP | ADR-006 |
| S08 | Docker Compose | Fog | Desplegar los servicios del fog | Servidores HP | ADR-012 |
| S09 | API Gateway + BFF móvil | Cloud | Entrada de la app y del panel web | AWS | Sección 6 |
| S10 | PostgreSQL (RDS) | Cloud | Datos consolidados del estudio | AWS | ADR-005 |
| S11 | S3 | Cloud | Modelos de IA y datasets | AWS | ADR-008 |
| S12 | Cognito | Cloud | Autenticación de usuarios | AWS | ADR-010 |
| S13 | Secrets Manager | Cloud | Guardar claves y contraseñas | AWS | ADR-010 |
| S14 | Entrenamiento y registro de modelos | Cloud | Entrenar y versionar la IA | AWS | ADR-008 |
| S15 | GitHub Actions | Transversal | CI/CD | GitHub | ADR-012 |
| S16 | OpenTelemetry + Prometheus | Transversal | Logs, métricas y trazas | Fog / AWS | ADR-011 |
| S17 | App Flutter + SQLite | Cliente | Uso sin conexión en el celular | Teléfono | ADR-007 |
| S18 | Firebase | Cliente | Solo notificaciones | Google | ADR-007 |
