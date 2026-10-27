# Grupo 1 Infraestructura y Servicios Cloud


AgriSmart Latacunga propone una solución de agricultura de precisión que integra dispositivos ESP32 y LoRaWAN, un nodo local fog, servicios de nube y una aplicación móvil Flutter con funcionamiento sin conexión. Para que el backend sea viable, primero se requiere identificar cada servicio, su función, dónde se ejecutará y de qué otros componentes depende. Este documento desarrolla el inventario a cargo del Grupo 1 en el hito H2. No constituye evidencia de despliegues ya realizados.

## 2. Objetivo y alcance

Objetivo: identificar, describir y priorizar los servicios de infraestructura, plataforma e integración necesarios para el backend de AgriSmart Latacunga, de modo que los demás equipos puedan clasificarlos, analizar su seguridad y justificar las decisiones arquitectónicas.

- Incluye API, cómputo, persistencia transaccional, archivos, identidad, integración IoT, mensajería, monitoreo, automatización, copias de seguridad y comunicaciones.
- Distingue servicios en AWS, software propio y componentes locales fog/edge.
- Registra estados verificables; no incluye aprovisionamiento, compras ni uso de credenciales de cuentas personales.

## 3. Referencia de la arquitectura

Flujo previsto: sensores ESP32/LoRaWAN → gateway y ChirpStack en fog → broker MQTT e ingesta → base de series temporales fog → sincronización segura → servicios cloud/API → aplicación Flutter y panel de investigación. La app conserva datos localmente mediante SQLite. Si Internet falla, la captura y el almacenamiento local deben continuar; la sincronización se reanuda al restablecerse la red.

Este flujo es una propuesta extraída de la versión 3 del proyecto y debe validarse con el equipo general. El backend cloud no recibe directamente cada lectura LoRaWAN: la arquitectura distingue claramente el gateway y el fog del procesamiento en AWS.

## 4. Inventario técnico de servicios

| ID | Componente | Tecnología candidata | Función | Ubicación | Estado |
|---|---|---|---|---|---|
| S01 | Cómputo backend | AWS EC2 o servicio administrado por decidir | Ejecutar API y tareas de aplicación | Cloud | Propuesto |
| S02 | API de aplicación | Servicio REST propio; OpenAPI | Exponer datos y operaciones de agricultores/parcelas | Cloud | Por desarrollar |
| S03 | Base de datos operacional | PostgreSQL en Amazon RDS | Persistir usuarios seudonimizados, parcelas y registros | Cloud | Propuesto |
| S04 | Objetos y respaldos | Amazon S3 | Guardar datasets, exportaciones, archivos y modelos | Cloud | Propuesto |
| S05 | Identidad y acceso | Amazon Cognito u OIDC equivalente | Autenticar usuarios y roles con control de acceso | Cloud | Propuesto |
| S06 | Red y protección | VPC, subredes, SG, TLS y DNS | Aislar componentes y proteger comunicaciones | Cloud | Por diseñar |
| S07 | Observabilidad cloud | CloudWatch; métricas y logs | Registrar errores, disponibilidad y alertas operativas | Cloud | Propuesto |
| S08 | CI/CD | GitHub Actions + repositorio | Automatizar pruebas y despliegues versionados | SaaS externo | Por confirmar |
| S09 | Notificaciones móviles | Firebase Cloud Messaging | Avisos y alertas cuando el teléfono tenga conectividad | Servicio externo | Propuesto |
| S10 | Correo opcional | Amazon SES u otro | Enviar mensajes de soporte o verificación si aplica | Cloud | Opcional |
| S11 | Gateway LoRaWAN | Gateway + ChirpStack | Concentrar tramas y gestionar dispositivos | Fog/local | Recurso por verificar |
| S12 | Mensajería fog | Broker MQTT | Distribuir eventos a consumidores locales | Fog/local | Propuesto |
| S13 | Series temporales fog | PostgreSQL/TimescaleDB | Guardar lecturas aun sin Internet | Fog/local | Propuesto |
| S14 | Sincronización | Servicio propio store-and-forward | Replicar lecturas de forma idempotente a nube | Fog↔cloud | Por desarrollar |
| S15 | Inferencia local | Servicio de modelos IA en fog | Producir recomendaciones disponibles sin Internet | Fog/local | Por validar |
| S16 | Datos móviles | SQLite en Flutter | Permitir consulta y cola de acciones offline | Dispositivo | Por desarrollar |

Nota: "Propuesto" significa identificado en el planteamiento; "Por desarrollar" no implica que exista; "Por confirmar" requiere evidencia institucional. Los nombres comerciales no equivalen a un inventario contratado.

## 5. Servicios mínimos y dependencias

| Prioridad | Servicios | Motivo | Condición para avanzar |
|---|---|---|---|
| P0 crítico | S01, S02, S03, S06 | Sin cómputo, API, datos y red segura no existe backend cloud operable. | Elegir alojamiento y contratos API/datos. |
| P1 necesario | S04, S05, S07, S08, S14 | Identidad, trazabilidad, archivos y sincronización confiable. | Definir permisos, políticas de backup y pruebas. |
| P2 complementario | S09, S10 | Mensajería externa condicionada al diseño de experiencia. | Confirmar casos de uso y costo. |
| Infraestructura local | S11, S12, S13, S15, S16 | Permite funcionamiento rural con desconexiones. | Coordinar con módulos fog, IoT y app. |

Dependencias clave: la API necesita conectividad privada a PostgreSQL, mecanismo de autenticación, secretos y registros; el sincronizador necesita identificación única por lectura, reintentos y reglas de resolución; la notificación push requiere dispositivo registrado y conexión de salida.

## 6. Mapa del backend por ubicación

| Zona | Servicios que operan | Interfaz principal | Responsabilidad del sistema |
|---|---|---|---|
| Parcela / edge | ESP32, sensores, radio LoRa, buffer local | Paquetes LoRaWAN | Medición y reenvío; calibración y energía. |
| Fog institucional/local | ChirpStack, MQTT, TimescaleDB, alertas, sincronizador | MQTT/HTTPS seguro | Persistencia durante cortes y envío posterior. |
| AWS / cloud | API, RDS, S3, identidad, monitoreo | HTTPS REST y réplica segura | Consolidación, acceso y administración central. |
| Cliente | Flutter, SQLite | HTTPS + caché local | Uso offline y sincronización eventual. |

## 7. Validación de recursos existentes

El documento fuente menciona servidores HP existentes y tecnologías planificadas, pero no aporta evidencia de que estén aprovisionadas para la clase. Este registro debe completarse con el responsable del curso antes de afirmar que un servicio existe.

| Recurso | Estado actual verificable | Evidencia que debe adjuntarse | Responsable |
|---|---|---|---|
| Repositorio GitHub del curso | No confirmado | URL del repositorio y captura de README | Coordinación H1 |
| Cuenta AWS del grupo | No confirmado | Identificador de cuenta parcial y roles, sin secretos | Coordinación cloud |
| Instancia EC2 / servicio API | No confirmado | Inventario de recursos; no mostrar IP privada sensible | Infraestructura |
| RDS PostgreSQL | No confirmado | Captura de estado y tipo de instancia, sin claves | Datos |
| Bucket S3 | No confirmado | Inventario de buckets y política revisada | Datos |
| Gateway LoRaWAN y servidores HP | Mencionados en propuesta; operación no comprobada | Acta/inventario institucional y prueba básica | Equipo IoT/fog |
| CI y monitoreo | No confirmado | Workflow y captura de ejecución o tablero | DevOps |

## 8. Seguridad y riesgos del inventario

| Riesgo | Control propuesto | Evidencia futura |
|---|---|---|
| Credenciales expuestas en repositorio | Usar roles, gestores de secretos y .gitignore; prohibir access keys en capturas. | Revisión de repositorio y política IAM. |
| Base de datos públicamente accesible | RDS en subred privada, SG restrictivos y cifrado. | Diagrama de red y configuración sin secretos. |
| Pérdida de lecturas ante cortes | Colas locales, reintentos idempotentes, backup y pruebas de reconexión. | Prueba de desconexión/reenvío. |
| Costos inesperados de AWS | Presupuesto, alertas, etiquetas y apagado de recursos no esenciales. | Captura de presupuesto y estimación. |
| Datos personales y geográficos | Acceso por roles, seudonimización, minimización y consentimiento. | Política de datos aprobada. |

Riesgo residual: las medidas propuestas reducen la exposición, pero no sustituyen auditorías de seguridad, verificación de respaldos ni pruebas en campo.

## 9. Actividades y evidencias del Grupo 1

| Paso | Actividad | Producto verificable |
|---|---|---|
| 1 | Revisar arquitectura y listar componentes | Inventario S01–S16 y alcance. |
| 2 | Validar dependencias y ubicación | Mapa por edge/fog/cloud/cliente. |
| 3 | Consultar recursos que realmente existen | Tabla de estados con evidencias o pendientes. |
| 4 | Registrar riesgos y acuerdos | Riesgos, controles y notas para equipos 2–4. |
| 5 | Entregar fragmento aprobado a docs/nube-H2.md | Commit y actualización del README. |
| 6 | Socializar a los otros equipos | Acta o comentario de revisión y cambios. |

Capturas sugeridas: estructura del repositorio, arquitectura del proyecto, inventario cloud obtenido sin secretos y commit que incorpora el apartado. No se deben fabricar capturas ni declarar recursos operativos sin validación.

## 10. Entrega a los otros equipos

| Equipo receptor | Información entregada | Decisión que deberá completar |
|---|---|---|
| Grupo 2 Modelos cloud | S01–S16, ubicación y tecnología candidata | Clasificar servicios concretos IaaS/PaaS/SaaS y justificar. |
| Grupo 3 Seguridad | Datos almacenados, interfaces, zonas y riesgos | Responsabilidades compartidas, identidad y manejo de secretos. |
| Grupo 4 Arquitectura | Dependencias P0/P1 y alternativas | Decisión versionada de alojamiento de API y base de datos. |

Acuerdo recomendado: el inventario funciona como lista base controlada por versiones. Ningún equipo debe sustituir un componente sin comunicarlo y actualizar las dependencias.

## 11. Conclusiones

- El backend AgriSmart exige distinguir infraestructura local fog y servicios en nube; esta separación responde a la conectividad rural intermitente.
- La base de la primera versión debe concentrarse en API, persistencia, identidad, red segura y sincronización confiable antes de incorporar servicios opcionales.
- El inventario es una propuesta técnica consistente con la versión 3; el estado real de cada recurso debe validarse con evidencia en el repositorio.
- La clasificación de servicios, responsabilidades y decisión definitiva de arquitectura corresponden a las siguientes estaciones y deben mantener trazabilidad con el inventario del Grupo 1.

.
