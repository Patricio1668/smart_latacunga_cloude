 # 🌾 AgriSmart Latacunga

**Evaluación del impacto de una aplicación móvil con inteligencia artificial en la optimización de la calidad del suelo y la productividad agrícola en Latacunga**

Plataforma IoT, Inteligencia Artificial y Computación Edge-Fog-Cloud diseñada para la monitorización edafológica y la toma de decisiones agrícolas en zonas rurales con conectividad intermitente.

---

## 📌 Tabla de Contenidos
- [Descripción General](#-descripción-general)
- [Arquitectura del Sistema](#-arquitectura-del-sistema)
- [Estado de la Arquitectura del Piloto](#-estado-de-la-arquitectura-del-piloto)
- [Módulos del Proyecto](#-módulos-del-proyecto)
- [Tecnologías Utilizadas](#-tecnologías-utilizadas)
- [Requisitos Previos](#-requisitos-previos)
- [Instalación y Configuración](#-instalación-y-configuración)
- [Estructura del Repositorio](#-estructura-del-repositorio)
- [Contribución](#-contribución)
- [Licencia y Contacto](#-licencia-y-contacto)

---

## 🌿 Descripción General

AgriSmart Latacunga abarca el diseño, desarrollo e implementación de un sistema integral que recopila datos de suelo (pH, humedad, temperatura y NPK) mediante nodos IoT ESP32 y sensores LoRaWAN. Utiliza procesamiento en la niebla (*fog computing*) para garantizar su operatividad *offline* en el sector rural y sincroniza con la nube (AWS) para el análisis avanzado e inferencia de modelos de Machine Learning.

---

## 🏗️ Arquitectura del Sistema

El proyecto sigue una arquitectura **Edge-Fog-Cloud** tolerante a fallos de conectividad:

1. **Edge (Parcela):** Nodos sensores ESP32 + LoRaWAN que recopilan métricas edafológicas cada 15 minutos.
2. **Fog (Local / UTC):** Servidor on-premise con ChirpStack, MQTT broker, TimescaleDB y motor de alertas e inferencia local.
3. **Cloud (AWS):** API y BFF en contenedores sobre ECS Fargate, balanceador de carga (ALB), base de datos PostgreSQL (RDS) y almacenamiento de modelos.
4. **Clientes:** Aplicación móvil multiplataforma (Flutter) funcional *offline-first* y Panel Web para investigadores.

---

## ☁️ Estado de la Arquitectura del Piloto

Este resumen facilita identificar las decisiones descritas para el piloto y dónde consultar su justificación. **Describe el diseño documentado; no afirma que los servicios ya estén desplegados.**

| Componente | Diseño documentado para el piloto | Detalle |
|---|---|---|
| **Fog** | Docker Compose, ChirpStack, MQTT y TimescaleDB para recibir y conservar lecturas sin conexión. | [Arquitectura y decisiones técnicas](docs/arquitectura/Arquitectura%20y%20decisiones%20t%C3%A9cnicas.md) |
| **API y BFF** | H3 y el documento de arquitectura describen contenedores en ECS Fargate, detrás de un ALB con HTTPS. | [H3: modelo de implementación](docs/nube-H3.md) |
| **Base de datos cloud** | PostgreSQL en RDS; el fog sincroniza con la API, no escribe directamente en la base de datos. | [H3: modelo de implementación](docs/nube-H3.md) |
| **Aplicación móvil** | Flutter con almacenamiento local SQLite para uso sin conexión. | [H2: arquitectura y decisiones](docs/nube-H2.md) |

**Puntos por armonizar:** H2 presenta API Gateway como elegido, mientras que H3 indica que no se usa en el piloto y describe un ALB delante de Fargate. También difieren en observabilidad: H2 describe OpenTelemetry, Prometheus y Grafana; H3 menciona CloudWatch y OpenTelemetry. Conviene acordar estas decisiones y actualizar los documentos antes de implementarlas o presentarlas como definitivas.

El [H2](docs/nube-H2.md) compara alternativas y desarrolla responsabilidades de seguridad; el [H3](docs/nube-H3.md) delimita el modelo de implementación del piloto. Cuando ambos documentos difieran, la decisión debe confirmarse con el equipo en lugar de asumir que una alternativa ya está aprobada.

---

## 🛠️ Módulos del Proyecto

El desarrollo está organizado en los siguientes módulos:

- `M1`: Captura IoT y Firmware ESP32
- `M2`: Red LoRaWAN y Gateway (ChirpStack)
- `M3`: Ingesta de Eventos (Broker MQTT)
- `M4`: Persistencia y Series de Tiempo (TimescaleDB / CQRS)
- `M5`: Motor de Alertas e Inferencia
- `M6`: Sincronización Fog-Nube
- `M7`: API y BFF en la nube, autenticación (OAuth 2.1 / OIDC)
- `M8`: APIs para Aliados (INIAP, GAD)
- `M9`: BFF Móvil y Aplicación Flutter
- `M10`: Observabilidad y Monitoreo (OpenTelemetry, Prometheus, Grafana)

---

## 📦 Entregables

| Entregable | Descripción | Archivo |
|---|---|---|
| **H1** | Fundamentos de Infraestructura en la Nube | [docs/nube-H1.md](docs/nube-H1.md) |
| **H2** | Arquitectura y Decisiones Técnicas | [docs/nube-H2.md](docs/nube-H2.md) |
| **H3** | Modelo de Implementación y Límites del Despliegue | [docs/nube-H3.md](docs/nube-H3.md) |

---

## 💻 Tecnologías Utilizadas

- **Lenguajes:** Python / Node.js / Dart (Flutter) / C++ (Arduino/ESP32)
- **Base de Datos:** PostgreSQL, TimescaleDB, SQLite local
- **IoT & Redes:** ESP32, LoRaWAN, ChirpStack, MQTT
- **IA & ML:** Scikit-Learn (Random Forest, XGBoost), TinyML, TensorFlow Lite
- **DevOps & Nube:** Docker, Docker Compose, AWS (ECS Fargate, ALB, S3, RDS), GitHub Actions

---

## ⚙️ Requisitos Previos

Asegúrate de tener instalados las siguientes herramientas en tu entorno local:

- [Git](https://git-scm.com/)
- [Docker Engine](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/install/)
- [Python 3.10+](https://www.python.org/)
- [Flutter SDK](https://docs.flutter.dev/get-started/install) *(para el módulo móvil)*

---

## 🚀 Instalación y Configuración

1. **Clonar el repositorio:**
   ```bash
   git clone [https://github.com/TU-USUARIO/agrismart-latacunga.git](https://github.com/TU-USUARIO/agrismart-latacunga.git)
   cd agrismart-latacunga
   un readme de lo que pidió el ingeniero
