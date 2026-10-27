# Proyecto AgriSmart Latacunga — Entregable H2

**Asignatura:** Computación en la Nube / Infraestructura, Seguridad y Gestión de Riesgo en las TI  
**Proyecto:** Evaluación del impacto de una aplicación móvil con inteligencia artificial en la optimización de la calidad del suelo y la productividad agrícola en Latacunga  

## 1. Resumen Arquitectónico y Criterio de Clasificación

La plataforma **AgriSmart Latacunga** implementa una arquitectura híbrida orientada a eventos **Edge–Fog–Cloud** con servicios hexagonales:

1. **Nivel Edge (Parcela):** 40 nodos fijos ESP32 + LoRa y 4 kits portátiles que capturan datos edafológicos (pH, humedad, temperatura y NPK) cada 15 minutos con almacenamiento local (*store-and-forward*) e inferencia ligera con TinyML.
2. **Nivel Fog (On-Premise en UTC / Parroquia 11 de Noviembre):** Servidores HP existentes que ejecutan servicios contenerizados para operar sin conexión a internet, garantizando $0\,\%$ de pérdida de datos.
3. **Nivel Cloud (AWS y servicios externos):** Servicios en la nube encargados de consolidar los datos del estudio cuasiexperimental (240 agricultores durante 12 meses), entrenar modelos de inteligencia artificial (Random Forest y XGBoost), gestionar identidades y exponer las APIs para la aplicación móvil Flutter, el panel web y los aliados institucionales (INIAP y GAD Latacunga).

---

## 2. Tabla de Clasificación IaaS / PaaS / SaaS del Inventario

A continuación se clasifica cada servicio del inventario del backend y de la infraestructura transversal del proyecto:

| # | Servicio / Componente | Capa | Modelo de Servicio | Proveedor / Tecnología | Función en AgriSmart Latacunga | Justificación Técnica de la Clasificación |
| :-: | :--- | :--- | :---: | :--- | :--- | :--- |
| 1 | **Instancias de Cómputo para Contenedores** | Cloud | **IaaS** | AWS EC2 + Docker | Ejecución de contenedores del *Backend for Frontend* (BFF) móvil, servicio receptor de sincronización y tareas de backend. | AWS provee la máquina virtual, red virtual (VPC) y almacenamiento de bloques; el equipo de AgriSmart administra el sistema operativo Linux, el motor de Docker y los contenedores. |
| 2 | **Almacenamiento de Objetos** | Cloud | **IaaS / PaaS** | Amazon S3 | Repositorio del registro de modelos de IA (*champion/challenger*), datasets edafológicos históricos y respaldos. | Aunque provee infraestructura de almacenamiento base (IaaS), opera técnica y operativamente como un servicio gestionado (PaaS): abstrae discos y sistemas de archivos y se consume vía API REST/SDK con escalabilidad automática. |
| 3 | **Base de Datos Relacional Consolidada** | Cloud | **PaaS (DBaaS)** | Amazon RDS (PostgreSQL) | Almacenamiento consolidado del estudio de impacto (línea base, seguimiento de 12 meses y postest) replicado desde el nodo fog (ADR-005). | AWS gestiona el aprovisionamiento del servidor, parches del sistema operativo, actualizaciones del motor PostgreSQL, alta disponibilidad y copias de seguridad automáticas; el equipo solo administra esquemas, consultas y datos. |
| 4 | **Pasarela de APIs** | Cloud | **PaaS** | Amazon API Gateway | Punto de entrada único para las APIs REST (OpenAPI) consumidas por la app móvil Flutter, el panel web de investigadores y los sistemas de INIAP y GAD (ADR-009, ADR-012). | El proveedor administra toda la infraestructura de enrutamiento, control de tráfico (*throttling*), caché y terminación TLS; el equipo únicamente define las rutas, políticas y contratos OpenAPI. |
| 5 | **Gestión de Identidad y Autenticación** | Cloud | **PaaS (IDaaS)** | Amazon Cognito | Autenticación y autorización con OAuth 2.1 / OIDC y PKCE para agricultores, investigadores y aliados externos (ADR-010). | Servicio gestionado de directorio de usuarios y emisión de tokens JWT. Elimina la necesidad de desplegar y mantener servidores de identidad propios, integrándose de forma nativa con API Gateway. |
| 6 | **Gestión de Secretos y Credenciales** | Cloud | **PaaS (SecaaS)** | AWS Secrets Manager | Resguardo cifrado y rotación de credenciales de base de datos, certificados mTLS y llaves de API (ADR-010). | Abstrae el hardware de seguridad (HSM) y la plataforma de bóveda de secretos; los servicios de AgriSmart consumen los secretos mediante llamadas autenticadas al SDK sin exponer variables en el código. |
| 7 | **Entrenamiento de Modelos de IA** | Cloud | **PaaS / CaaS** | Contenedores en AWS + S3 | Entrenamiento y validación de modelos predictivos (exactitud $\ge 85\,\%$ en clasificación y $R^2 \ge 0.80$ en regresión) antes de su descarga al nodo fog (ADR-008). | Se ejecuta bajo demanda sobre entornos contenerizados conectándose al catálogo de datos en RDS y guardando los artefactos entrenados en S3 para su despliegue controlado. |
| 8 | **Servicio de Notificaciones Móviles** | Cloud Externa | **PaaS (BaaS)** | Firebase Cloud Messaging (FCM) | Envío de notificaciones *push* y alertas agronómicas a los dispositivos Android de los agricultores (ADR-007). | Funciona como un *Backend as a Service* (BaaS): Google administra toda la plataforma de colas y entrega a dispositivos Android, mientras que el backend de AgriSmart solo invoca la API de envío. |
| 9 | **Control de Versiones y CI/CD** | Transversal | **SaaS** | GitHub + GitHub Actions | Alojamiento de código fuente, contratos API-First (OpenAPI / AsyncAPI) y tuberías de integración y despliegue continuo (ADR-012). | Es una aplicación completa consumida a través de la web. El equipo de desarrollo no gestiona servidores Git ni nodos de automatización propios; solo configura los flujos de trabajo (`.yml`). |
| 10 | **Nodo Fog (Gateway LoRaWAN, MQTT, Ingesta, IA Local, TimescaleDB, Observabilidad)** | Fog (Borde) | **On-Premise (CaaS local)** | Servidores HP existentes + Docker Compose | Recepción LoRaWAN (ChirpStack), broker MQTT, persistencia de series de tiempo (TimescaleDB), inferencia local (RF/XGBoost), sincronización fog–nube y monitoreo con OpenTelemetry/Prometheus/Grafana (ADR-001 a ADR-012). | Se ejecuta en infraestructura física propia de la UTC o de la parroquia 11 de Noviembre. Usa virtualización ligera por contenedores (Docker Compose) para garantizar operación autónoma cuando falla el enlace a internet. |

---

## 3. Justificación Técnica de la Distribución de Modelos

La elección combinada de **On-Premise (Fog) + IaaS + PaaS + SaaS** responde a cuatro restricciones reales del proyecto:

1. **Restricción presupuestaria estricta ($85\text{ USD/mes}$ para nube):**
   El presupuesto total para servicios AWS durante los 24 meses del proyecto es de $2\,000\text{ USD}$. El uso de servicios PaaS de pago por uso (API Gateway, Cognito, S3) junto con instancias acotadas de RDS y EC2 mantiene el costo mensual dentro del límite, evitando pagar licencias SaaS empresariales o clústeres sobredimensionados.
2. **Equipo pequeño y rotación de estudiantes (Módulos M1–M10):**
   Los estudiantes de Sistemas de Información y Software cumplen 240 horas de prácticas preprofesionales y rotan por periodos académicos. Según el **ADR-002** y el **ADR-012**, delegar la administración de la base de datos (RDS), la identidad (Cognito) y la pasarela de APIs (API Gateway) al modelo **PaaS** reduce la carga operativa y evita caídas por errores de configuración manual de servidores.
3. **Tolerancia a la desconexión rural (Riesgo crítico):**
   Una arquitectura $100\,\%$ en la nube pública (IaaS/PaaS puro) fracasaría ante los cortes de internet de la zona rural de Latacunga. Por ello, el **ADR-001** mantiene la ingesta crítica, el broker MQTT y TimescaleDB en el **Nodo Fog local**, usando la nube pública únicamente para consolidación asíncrona (mediante el sincronizador con *retry* y *circuit breaker*), entrenamiento pesado y acceso externo.
4. **Cumplimiento de la LOPDP y Seguridad (ADR-010):**
   El uso de servicios PaaS especializados en seguridad (Amazon Cognito con OAuth 2.1/PKCE y AWS Secrets Manager) permite aplicar cifrado en tránsito (TLS/mTLS), cifrado en reposo y seudonimización de los datos personales de los 240 agricultores sin construir criptografía ni sistemas de autenticación desde cero.

---

## 4. Comparativa Técnica de Alternativas por Subsistema

Para sustentar por qué se eligió cada modelo en el inventario frente a otras opciones posibles, se presenta la comparativa de alternativas técnicas:

### 4.1. Base de Datos Relacional en la Nube (Consolidación del Estudio)

| Criterio | Alternativa IaaS: PostgreSQL en AWS EC2 | Alternativa PaaS: Amazon RDS para PostgreSQL *(Elegida)* | Alternativa SaaS / BaaS: Firebase Firestore |
| :--- | :--- | :--- | :--- |
| **Administración** | El equipo instala PostgreSQL, configura discos EBS, parches de SO y scripts de *backup*. | AWS automatiza parches, respaldos diarios, *snapshots* y recuperación puntual. | Totalmente gestionado por Google, sin servidores visibles. |
| **Adecuación al estudio** | Alta compatibilidad SQL para análisis estadístico (DID y PSM), pero alto riesgo de pérdida por error humano. | Excelente soporte relacional para cruzar línea base, sensores y postest sin riesgo operativo. | Modelo documental NoSQL que dificulta consultas relacionales complejas y cruces estadísticos. |
| **Alineación ADR** | Contradice el ADR-012 (preferencia por servicios administrados en AWS). | **Cumple ADR-005 y ADR-012.** | Descartado por **ADR-007** (Firebase se limita exclusivamente a notificaciones). |

### 4.2. Ingesta IoT y Base de Datos de Series de Tiempo

| Criterio | Alternativa On-Premise Contenerizada: ChirpStack + MQTT + TimescaleDB en Nodo Fog *(Elegida)* | Alternativa PaaS Cloud: AWS IoT Core + Amazon Timestream | Alternativa SaaS IoT: Ubidots / ThingSpeak Comercial |
| :--- | :--- | :--- | :--- |
| **Operación sin internet** | **Total ($100\,\%$ funcional offline).** Captura, almacena y genera alertas localmente en la parroquia o UTC. | **Nula.** Si cae el enlace de internet rural, el gateway no puede enviar tramas a la nube. | **Nula.** Requiere conectividad continua hacia los servidores del proveedor SaaS. |
| **Costo a 24 meses** | Costo de licencia $0\text{ USD}$ (software abierto sobre servidores HP existentes). | Costo variable por millón de mensajes e ingesta continua de 40 nodos cada 15 minutos. | Licencias mensuales elevadas al superar el límite de dispositivos y variables por nodo. |
| **Alineación ADR** | **Cumple ADR-001, ADR-003, ADR-004 y ADR-005.** | Incumple el requisito de resiliencia ante desconexión rural. | Incumple presupuesto y soberanía de datos del estudio. |

### 4.3. Identidad, Autenticación y Autorización

| Criterio | Alternativa IaaS: Keycloak en Docker sobre EC2 | Alternativa PaaS: Amazon Cognito *(Elegida)* | Alternativa SaaS: Auth0 Plan Comercial |
| :--- | :--- | :--- | :--- |
| **Consumo de recursos** | Alto consumo de memoria RAM en la instancia EC2 (Java/Quarkus) y mantenimiento manual. | Sin consumo de cómputo propio; totalmente gestionado por AWS. | Sin consumo de cómputo propio; gestionado en la nube de Auth0. |
| **Integración y Costo** | Gratuito en licencia, pero encarece el tamaño de la instancia EC2 requerida. | Cubierto por la capa gratuita de AWS para el volumen del proyecto (~240 agricultores + investigadores) e integración nativa con API Gateway. | Límites en planes gratuitos para roles avanzados y costo elevado al escalar a planes de pago. |
| **Alineación ADR** | Mayor carga de administración para los estudiantes. | **Cumple ADR-010 y ADR-012** (OAuth 2.1 / OIDC con PKCE integrado en AWS). | Innecesario existiendo integración nativa en el mismo proveedor cloud (AWS). |

### 4.4. Exposición de APIs y Ejecución de Servicios Backend

| Criterio | Alternativa IaaS Pura: Nginx + Contenedores en EC2 | Alternativa Híbrida PaaS + Contenedores: API Gateway + Servicios en Docker *(Elegida)* | Alternativa PaaS Serverless Pura: AWS Lambda para todo el backend |
| :--- | :--- | :--- | :--- |
| **Portabilidad Fog–Cloud** | Alta, pero obliga a gestionar manualmente certificados TLS, *rate limiting* y seguridad perimetral en Nginx. | **Óptima:** API Gateway resuelve la seguridad perimetral y contratos OpenAPI (PaaS), mientras que los contenedores mantienen el mismo empaquetado Docker entre el fog y la nube. | Baja portabilidad hacia el nodo fog local y posibles arranques en frío (*cold starts*) en procesos de sincronización por lotes. |
| **Alineación ADR** | Mayor riesgo en seguridad perimetral para las APIs de aliados (INIAP/GAD). | **Cumple ADR-002, ADR-009 y ADR-012.** | Dificulta la simetría de ejecución con los servicios contenerizados del nodo fog. |

### 4.5. Observabilidad y Monitoreo de la Plataforma

| Criterio | Alternativa Contenerizada: OpenTelemetry + Prometheus + Grafana *(Elegida)* | Alternativa PaaS: Amazon CloudWatch + AWS Managed Grafana | Alternativa SaaS: Datadog / New Relic |
| :--- | :--- | :--- | :--- |
| **Cobertura Híbrida** | Funciona de manera idéntica dentro del nodo fog (sin internet) y en la nube, recolectando métricas, logs y trazas. | Orientado a recursos dentro de AWS; no permite visualizar métricas localmente en el nodo fog si se corta el internet. | Requiere salida permanente a internet para enviar telemetría a la nube del proveedor SaaS. |
| **Impacto en Presupuesto** | **$0\text{ USD}$ en licencias**, ejecutándose como contenedores dentro de la infraestructura existente. | El cobro por métricas personalizadas, ingesta de logs y usuarios de AWS Managed Grafana compromete los $85\text{ USD/mes}$. | Excede ampliamente el presupuesto asignado para nube. |
| **Alineación ADR** | **Cumple estrictamente el ADR-011.** | Descartado por costo y dependencia de internet. | Descartado por costo de licenciamiento. |

---

## 5. Conclusiones para la Integración del H2

1. **Predominio estratégico de PaaS en la nube pública:** En la capa Cloud (AWS), el modelo **PaaS** (Amazon RDS, API Gateway, Cognito, Secrets Manager, S3) es la opción técnica más adecuada para AgriSmart Latacunga porque maximiza la disponibilidad y la seguridad reduciendo al mínimo la carga administrativa del equipo docente y estudiantil, manteniéndose dentro del presupuesto de $85\text{ USD/mes}$.
2. **Contenerización sobre IaaS y On-Premise para portabilidad:** El uso de contenedores Docker (sobre servidores HP en el nivel Fog y sobre instancias de cómputo en AWS) permite mantener una arquitectura basada en servicios y eventos (**ADR-002**) altamente portable sin caer en la complejidad operativa de un clúster de microservicios ni en el bloqueo de proveedor (*vendor lock-in*) en la lógica de negocio e inferencia de IA.
3. **Adopción selectiva de SaaS/BaaS:** El uso de **GitHub Actions (SaaS)** para CI/CD y **Firebase Cloud Messaging (BaaS/PaaS)** exclusivamente para notificaciones *push* resuelve necesidades transversales críticas sin costo financiero ni esfuerzo de mantenimiento de infraestructura adicional.
