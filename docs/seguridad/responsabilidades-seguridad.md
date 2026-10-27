r#  Responsabilidades y seguridad

**Proyecto:** Nube inteligente de Latacunga
**Equipo 3** · Responsabilidades y seguridad

> 📥 Parte de: Equipo 1 (inventario de servicios) y Equipo 2 (clasificación cloud)
> 📤 Alimenta a: Equipo 4 (decisiones de arquitectura)

---

## 📌 Resumen ejecutivo

El presente documento establece la distribución de responsabilidades entre el proveedor de servicios en la nube y el equipo del proyecto **Nube inteligente de Latacunga**, conforme al modelo de responsabilidad compartida, con el fin de garantizar que cada componente de la solución cuente con un responsable claramente definido.

Su alcance comprende los cuatro componentes principales del sistema: **backend, base de datos, infraestructura y pruebas**. Para cada uno se determina qué aspectos de seguridad son cubiertos por el proveedor y cuáles corresponden al equipo, tales como la gestión de accesos, la protección de credenciales, la configuración de la base de datos, los respaldos y la actualización de dependencias.

Adicionalmente, se identifican los riesgos más relevantes, se definen los controles de seguridad mínimos con su nivel de prioridad y se formulan recomendaciones técnicas dirigidas al **Equipo 4**, de modo que sirvan como insumo para la toma de decisiones de arquitectura y`nube-H2.md`.

---

## 1. 🎯 Objetivo general

Evaluar las responsabilidades, los riesgos y los controles de seguridad en los servicios cloud, con el fin de proponer recomendaciones que permitan al Equipo 4 tomar decisiones adecuadas para el diseño de una arquitectura segura.

Objetivos específicos

1. Identificar las responsabilidades del proveedor cloud y del equipo encargado de administrar los servicios y datos.

2. Analizar los principales riesgos de seguridad asociados al uso de servicios en la nube, como el acceso no autorizado, la pérdida de información y las configuraciones incorrectas.

3. Proponer controles de seguridad adecuados para prevenir, detectar y reducir los riesgos identificados.

4. Establecer recomendaciones técnicas que orienten al Equipo 4 en la toma de decisiones para diseñar una arquitectura cloud segura y confiable.

---

## 2. 🤝 Modelo de responsabilidad compartida

En la nube, la seguridad se reparte entre el proveedor y quien usa el servicio.

| Capa | ☁️ Proveedor cloud | 👥 Nuestro equipo |
|------|:------------------:|:-----------------:|
| Centros de datos y hardware | ✅ | |
| Red física y virtualización | ✅ | |
| Sistema operativo (si es IaaS) | | ✅ |
| Aplicación (backend) | | ✅ |
| Base de datos: configuración y accesos | | ✅ |
| Datos y su clasificación | | ✅ |
| Usuarios, roles y contraseñas | | ✅ |

**Regla práctica:** el proveedor protege la nube; nosotros protegemos lo que ponemos **dentro** de la nube.

---

## 3. 🧩 Responsabilidades por servicio

| Servicio | Tipo | ☁️ Proveedor | 👥 Equipo |
|----------|------|--------------|-----------|
| **Backend** | PaaS / IaaS | Servidores, red y disponibilidad de la plataforma | Código, dependencias, configuración y despliegue |
| **Base de datos H2** | Embebida / IaaS (si va en VM) | Almacenamiento y VM (si aplica) | Esquema, contraseñas, respaldos y deshabilitar consola H2 en prod.
| **Infraestructura** | IaaS | Hardware, red física y virtualización | Red, firewall, sistema operativo y parches |
| **Pruebas** | Entorno propio | Ninguna (si se ejecutan localmente) | Casos de prueba, datos de prueba y ejecución |

> ℹ️ Los tipos (PaaS/IaaS) deben coincidir con lo definido por los Equipos 1 y 2 en `nube-H1.md` y `nube-H2.md`.

### 3.1 Responsabilidades según el modelo de servicio cloud

Las responsabilidades de seguridad dependen del modelo de servicio contratado. Cuanto mayor sea el nivel de administración que proporciona el proveedor, menor será la cantidad de componentes de infraestructura que deberá gestionar directamente el equipo. Sin embargo, el equipo mantiene responsabilidades sobre los datos, las identidades y las configuraciones que controla.

| Elemento | IaaS | PaaS | SaaS |
|---|---|---|---|
| Centro de datos y hardware | Proveedor | Proveedor | Proveedor |
| Virtualización | Proveedor | Proveedor | Proveedor |
| Sistema operativo | Equipo | Proveedor | Proveedor |
| Actualizaciones del sistema operativo | Equipo | Proveedor | Proveedor |
| Plataforma de ejecución | Equipo | Proveedor | Proveedor |
| Código de la aplicación | Equipo | Equipo | Proveedor |
| Configuración de la aplicación | Equipo | Equipo | Compartida, según el servicio |
| Gestión de usuarios y permisos | Equipo | Equipo | Equipo, según las funciones disponibles |
| Protección y clasificación de datos | Equipo | Equipo | Equipo |
| Seguridad física de la infraestructura | Proveedor | Proveedor | Proveedor |

## 4. 🛡️ Controles de seguridad

Los siguientes controles constituyen el **mínimo exigible** para los componentes del proyecto. Se clasifican por prioridad: **Alta** (implementar antes de cualquier despliegue) y **Media** (implementar durante la operación).

### 4.1 Gestión de identidades y accesos

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Mínimo privilegio | Cada usuario y servicio recibe solo los permisos que necesita | Todos | Equipo | Alta |
| Roles por tipo de usuario | Separar perfiles (administrador, operador, consulta) | Backend | Equipo | Alta |
| Autenticación robusta | Contraseñas seguras y, cuando sea posible, doble factor (MFA) en cuentas administrativas | Backend / Infraestructura | Equipo | Alta |
| Revisión periódica de permisos | Revisar y retirar accesos que ya no se necesiten | Todos | Equipo | Media |

### 4.2 Protección de secretos

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Secretos fuera del repositorio | Contraseñas, claves y tokens bloqueados con `.gitignore` | Repositorio | Equipo | Alta |
| Variables de entorno | Las credenciales se leen del entorno, nunca del código | Backend | Equipo | Alta |
| Rotación de credenciales | Cambiar claves periódicamente y ante cualquier sospecha de filtración | Todos | Equipo | Media |

### 4.3 Protección de datos

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Cifrado en tránsito | Toda comunicación mediante HTTPS (TLS) | Backend | Equipo | Alta |
| Cifrado en reposo | Activar el cifrado del almacenamiento cuando el servicio lo permita | Base de datos | Proveedor / Equipo | Media |
| Clasificación de datos | Identificar qué datos son personales o sensibles y tratarlos según su nivel | Base de datos | Equipo | Media |
| Respaldos periódicos | Copias automáticas de la base de datos, con prueba de restauración | Base de datos | Equipo | Media |

### 4.4 Base de datos


| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Consola H2 protegida | Deshabilitada fuera del entorno de desarrollo | Base de datos | Equipo | Alta |
| Credenciales propias | No usar el usuario y contraseña por defecto | Base de datos | Equipo | Alta |
| Acceso restringido por red | La base de datos no debe ser accesible desde internet | Infraestructura | Equipo | Alta |

### 4.5 Red e infraestructura

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Firewall y puertos mínimos | Abrir solo los puertos necesarios (por ejemplo, 443) | Infraestructura | Equipo | Alta |
| Actualizaciones y parches | Mantener sistema operativo y servicios al día | Infraestructura | Equipo | Media |
| Separación de entornos | Desarrollo, pruebas y producción independientes | Infraestructura | Equipo | Media |

### 4.4 Monitoreo y mantenimiento

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Registros (logs) | Guardar accesos y errores, sin incluir datos sensibles | Backend | Equipo | Media |
| Alertas de seguridad | Notificar intentos de acceso fallidos o actividad inusual | Backend / Infraestructura | Equipo | Media |
| Actualización de dependencias | Revisar y actualizar librerías con regularidad | Backend | Equipo | Media |
| Datos de prueba | Usar datos ficticios en pruebas, nunca datos reales | Pruebas | Equipo | Media |

### 4.7 Cumplimiento normativo y legal

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Privacidad por diseño | Recolectar solo los datos estrictamente necesarios de los ciudadanos. | Backend | Equipo | Alta |
| Registro de auditoría | Mantener bitácoras inalterables de quién accede o modifica datos personales. | Base de datos | Equipo | Alta |


---

## 5. ⚠️ Riesgos identificados

| # | Riesgo | Impacto | Mitigación |
|---|--------|:-------:|------------|
| 1 | Credenciales subidas al repositorio | 🔴 Alto | `.gitignore` y revisión antes de cada commit |
| 2 | Base de datos expuesta | 🔴 Alto | Restringir red y accesos; deshabilitar consola H2 |
| 3 | Pérdida de datos | 🟠 Medio | Respaldos periódicos |
| 4 | Dependencias desactualizadas | 🟠 Medio | Actualizar y revisar librerías con regularidad |

---

## 6. 💡 Recomendaciones para el Equipo 4

1. **Base de datos:** usar H2 solo en desarrollo y pruebas. Para producción, considerar una base de datos administrada (PaaS) que reduzca la carga de respaldos y parches.
2. **Secretos:** definir desde el inicio dónde se guardan (variables de entorno o gestor de secretos del proveedor).
3. **Red:** exponer el backend solo por HTTPS y limitar los puertos abiertos.
4. **Entornos:** separar desarrollo, pruebas y producción.
5. **Accesos:** documentar quién tiene permisos de administración y revisarlos periódicamente.
6. **Cumplimiento legal (LOPDP):** Seleccionar infraestructura que facilite el cumplimiento de la normativa ecuatoriana (derecho al olvido y resguardo seguro de datos).
7. **Base de datos de producción:** Reemplazar H2 por un motor relacional en modalidad PaaS (ej. PostgreSQL o MySQL) para delegar parches y respaldos al proveedor.
8. **DevSecOps:** Integrar herramientas de análisis estático (SAST) en GitLab para bloquear commits que contengan credenciales o vulnerabilidades.
9. **APIs seguras:** Exigir tokens JWT de corta duración y aplicar limitación de tasa (Rate Limiting) para evitar ataques de denegación de servicio.
10. **Plan de incidentes:** Establecer un protocolo rápido de notificación y aislamiento del servicio en caso de detectar una brecha de datos.
11. 

---

## ✅ Conclusión

La seguridad del proyecto es **compartida**: el proveedor cubre la infraestructura base y nuestro equipo es responsable de la aplicación, los datos, los accesos y la configuración. Aplicar los controles de este documento reduce los riesgos más críticos.
