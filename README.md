 # 🌾 AgriSmart Latacunga

**Evaluación del impacto de una aplicación móvil con inteligencia artificial en la optimización de la calidad del suelo y la productividad agrícola en Latacunga**

Plataforma IoT, Inteligencia Artificial y Computación Edge-Fog-Cloud diseñada para la monitorización edafológica y la toma de decisiones agrícolas en zonas rurales con conectividad intermitente.

---

## 📌 Tabla de Contenidos
- [Descripción General](#-descripción-general)
- [Arquitectura del Sistema](#-arquitectura-del-sistema)
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
3. **Cloud (AWS):** API Gateway, base de datos PostgreSQL (RDS), almacenamiento de modelos y servicios analíticos.
4. **Clientes:** Aplicación móvil multiplataforma (Flutter) funcional *offline-first* y Panel Web para investigadores.

---

## 🛠️ Módulos del Proyecto

El desarrollo está organizado en los siguientes módulos:

- `M1`: Captura IoT y Firmware ESP32
- `M2`: Red LoRaWAN y Gateway (ChirpStack)
- `M3`: Ingesta de Eventos (Broker MQTT)
- `M4`: Persistencia y Series de Tiempo (TimescaleDB / CQRS)
- `M5`: Motor de Alertas e Inferencia
- `M6`: Sincronización Fog-Nube
- `M7`: API Gateway y Autenticación (OAuth 2.1)
- `M8`: APIs para Aliados (INIAP, GAD)
- `M9`: BFF Móvil y Aplicación Flutter
- `M10`: Observabilidad y Monitoreo (OpenTelemetry, Prometheus, Grafana)

---

## 💻 Tecnologías Utilizadas

- **Lenguajes:** Python / Node.js / Dart (Flutter) / C++ (Arduino/ESP32)
- **Base de Datos:** PostgreSQL, TimescaleDB, SQLite local
- **IoT & Redes:** ESP32, LoRaWAN, ChirpStack, MQTT
- **IA & ML:** Scikit-Learn (Random Forest, XGBoost), TinyML, TensorFlow Lite
- **DevOps & Nube:** Docker, Docker Compose, AWS (EC2, S3, RDS), GitHub Actions

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
