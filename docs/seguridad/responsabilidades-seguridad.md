# Responsabilidades y seguridad

**Proyecto:** Nube inteligente de Latacunga  
**Equipo 3** · Responsabilidades y seguridad

> 📥 Parte de: Equipo 1 (inventario de servicios) y Equipo 2 (clasificación cloud)  
> 📤 Alimenta a: Equipo 4 (decisiones de arquitectura)

---

## 📌 Resumen ejecutivo

El presente documento establece la distribución de responsabilidades entre el proveedor de servicios en la nube y el equipo del proyecto **Nube inteligente de Latacunga**, conforme al modelo de responsabilidad compartida, con el fin de garantizar que cada componente de la solución cuente con un responsable claramente definido.

Su alcance comprende los cuatro componentes principales del sistema: **backend, base de datos, infraestructura y pruebas**. Para cada uno se determina qué aspectos de seguridad son cubiertos por el proveedor y cuáles corresponden al equipo, tales como la gestión de accesos, la protección de credenciales, la configuración de la base de datos, los respaldos y la actualización de dependencias.

Adicionalmente, se identifican los riesgos más relevantes, se definen los controles de seguridad mínimos con su nivel de prioridad, se propone una hoja de ruta de implementación y se formulan recomendaciones técnicas dirigidas al **Equipo 4**, de modo que sirvan como insumo para la toma de decisiones de arquitectura y para la coherencia con lo definido en `nube-H1.md` y `nube-H2.md`.

---

## 1. 🎯 Objetivo general

Evaluar las responsabilidades, los riesgos y los controles de seguridad en los servicios cloud, con el fin de proponer recomendaciones que permitan al Equipo 4 tomar decisiones adecuadas para el diseño de una arquitectura segura.

**Objetivos específicos:**

1. Identificar las responsabilidades del proveedor cloud y del equipo encargado de administrar los servicios y datos.
2. Analizar los principales riesgos de seguridad asociados al uso de servicios en la nube, como el acceso no autorizado, la pérdida de información y las configuraciones incorrectas.
3. Proponer controles de seguridad adecuados para prevenir, detectar y reducir los riesgos identificados.
4. Establecer recomendaciones técnicas que orienten al Equipo 4 en la toma de decisiones para diseñar una arquitectura cloud segura y confiable.

---

## 2. Modelo de responsabilidad compartida

En la nube, la seguridad se reparte entre el proveedor y quien usa el servicio.

| Capa | Proveedor cloud | 👥 Nuestro equipo |
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
| **Base de datos H2** | Embebida / IaaS (si va en VM) | Almacenamiento y VM (si aplica) | Esquema, contraseñas, respaldos y deshabilitar consola H2 en prod. |
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

---

## 4. 🛡️ Controles de seguridad

Los siguientes controles constituyen el **mínimo exigible** para los componentes del proyecto. Se clasifican por prioridad: **Alta** (implementar antes de cualquier despliegue) y **Media** (implementar durante la operación). El criterio de priorización se explica en la sección 6.

### 4.1 Gestión de identidades y accesos

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Mínimo privilegio | Cada usuario y servicio recibe solo los permisos que necesita | Todos | Equipo | Alta |
| Roles por tipo de usuario | Separar perfiles (administrador, operador, consulta) | Backend | Equipo | Alta |
| Autenticación robusta | Contraseñas seguras y, cuando sea posible, doble factor (MFA) en cuentas administrativas | Backend / Infraestructura | Equipo | Alta |
| Revisión periódica de permisos | Revisar y retirar accesos que ya no se necesiten, incluidos los de personas que dejan el proyecto | Todos | Equipo | Media |
| Cuentas administrativas separadas | Usar cuentas distintas para administrar y para el trabajo diario; evitar cuentas compartidas | Todos | Equipo | Media |

### 4.2 Protección de secretos

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Secretos fuera del repositorio | Contraseñas, claves y tokens bloqueados con `.gitignore` | Repositorio | Equipo | Alta |
| Variables de entorno | Las credenciales se leen del entorno, nunca del código | Backend | Equipo | Alta |
| Rotación de credenciales | Cambiar claves periódicamente y ante cualquier sospecha de filtración | Todos | Equipo | Media |
| Gestor de secretos | Evaluar el gestor de secretos del proveedor para producción en lugar de archivos de configuración | Backend / Infraestructura | Equipo | Media |

### 4.3 Protección de datos

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Cifrado en tránsito | Toda comunicación mediante HTTPS (TLS) | Backend | Equipo | Alta |
| Cifrado en reposo | Activar el cifrado del almacenamiento cuando el servicio lo permita. Sube a Alta porque el sistema trata datos personales de ciudadanos | Base de datos | Proveedor / Equipo | Alta |
| Clasificación de datos | Identificar qué datos son personales o sensibles y tratarlos según su nivel | Base de datos | Equipo | Media |
| Respaldos periódicos | Copias automáticas de la base de datos, con prueba de restauración (ver 4.10) | Base de datos | Equipo | Media |

### 4.4 Base de datos

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Consola H2 protegida | Deshabilitada fuera del entorno de desarrollo (por ejemplo, `spring.h2.console.enabled=false` si se usa Spring) | Base de datos | Equipo | Alta |
| Credenciales propias | No usar el usuario y contraseña por defecto | Base de datos | Equipo | Alta |
| Acceso restringido por red | La base de datos no debe ser accesible desde internet | Infraestructura | Equipo | Alta |
| Archivo de base protegido | Si H2 corre en modo embebido, el archivo de la base no debe quedar en un directorio accesible ni versionado en el repositorio | Base de datos | Equipo | Alta |

> ℹ️ **Nota sobre H2:** en modo embebido la base vive dentro de la aplicación y comparte su ciclo de vida; en modo servidor acepta conexiones por red y requiere restringir el puerto. En ambos casos se recomienda limitar su uso a desarrollo y pruebas (ver recomendación 1).

### 4.5 Red e infraestructura

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Firewall y puertos mínimos | Abrir solo los puertos necesarios (por ejemplo, 443) | Infraestructura | Equipo | Alta |
| Acceso administrativo seguro | Administrar por VPN o servidor bastión; no dejar SSH/RDP abierto a internet | Infraestructura | Equipo | Alta |
| Segmentación de red | Subred pública solo para el punto de entrada; backend y base de datos en subredes privadas | Infraestructura | Equipo | Media |
| WAF y protección DDoS | Usar el firewall de aplicaciones web y la protección contra denegación de servicio que ofrezca el proveedor | Infraestructura | Proveedor / Equipo | Media |
| Actualizaciones y parches | Mantener sistema operativo y servicios al día | Infraestructura | Equipo | Media |
| Separación de entornos | Desarrollo, pruebas y producción independientes | Infraestructura | Equipo | Media |

### 4.6 Monitoreo y mantenimiento

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
| Inventario de datos personales | Documentar qué datos personales y sensibles maneja el proyecto, para qué se usan y dónde se almacenan | Base de datos | Equipo | Alta |
| Base legal y consentimiento | Definir la base legal del tratamiento y, cuando corresponda, obtener y registrar el consentimiento del titular | Backend | Equipo | Alta |
| Derechos de los titulares | Poder atender solicitudes de acceso, rectificación, eliminación (derecho al olvido), oposición y portabilidad | Backend / Base de datos | Equipo | Media |
| Notificación de brechas | Definir el procedimiento y los plazos para avisar a la autoridad de protección de datos y a los afectados | Todos | Equipo | Alta |
| Ubicación de los datos | Verificar la región donde el proveedor almacena los datos y si hay transferencia internacional, según la LOPDP | Infraestructura | Equipo | Media |

### 4.8 Seguridad del backend y las APIs

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Validación de entradas | Validar y sanear todos los datos recibidos; usar consultas parametrizadas contra inyección SQL | Backend | Equipo | Alta |
| Protección contra XSS y CSRF | Codificar salidas y aplicar protecciones anti-CSRF cuando haya sesiones | Backend | Equipo | Alta |
| Tokens y sesiones | JWT de corta duración, expiración y renovación controlada | Backend | Equipo | Alta |
| Limitación de tasa | Aplicar rate limiting para mitigar abuso y denegación de servicio | Backend | Equipo | Media |
| Configuración de CORS | Permitir solo los orígenes necesarios | Backend | Equipo | Media |
| Cabeceras de seguridad | Activar cabeceras como HSTS, Content-Security-Policy y X-Content-Type-Options | Backend | Equipo | Media |
| Manejo de errores | No exponer trazas ni detalles internos al usuario final | Backend | Equipo | Media |

### 4.9 Seguridad en el ciclo de desarrollo (DevSecOps)

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Revisión de código | Merge requests obligatorios con al menos una aprobación | Repositorio | Equipo | Alta |
| Protección de ramas | Bloquear envíos directos a la rama principal | Repositorio | Equipo | Alta |
| Análisis estático (SAST) | Integrar en GitLab para detectar credenciales y vulnerabilidades antes del merge | Repositorio / CI | Equipo | Media |
| Escaneo de dependencias | Revisar librerías con herramientas como OWASP Dependency-Check | Backend / CI | Equipo | Media |
| Secretos en el pipeline | Guardar las credenciales de CI/CD como variables protegidas, no en archivos | CI/CD | Equipo | Alta |

### 4.10 Respaldos y continuidad del servicio

| Control | Descripción | Componente | Responsable | Prioridad |
|---------|-------------|------------|-------------|:---------:|
| Definir RPO y RTO | Acordar cuánta pérdida de datos (RPO) y cuánto tiempo de caída (RTO) son tolerables | Todos | Equipo | Media |
| Frecuencia de respaldos | Establecer la frecuencia según el RPO (por ejemplo, diaria) y la retención de copias | Base de datos | Equipo | Media |
| Almacenamiento separado | Guardar copias en otra zona o región y con acceso restringido, para resistir borrado o ransomware | Base de datos | Equipo | Media |
| Prueba de restauración | Probar la recuperación al menos una vez por periodo y documentar el resultado | Base de datos | Equipo | Media |

### 4.11 Respuesta a incidentes

Se recomienda un protocolo sencillo de cinco fases:

| Fase | Acción | Responsable sugerido |
|------|--------|----------------------|
| 1. Detección | Identificar la alerta o el reporte y confirmar que es un incidente | Quien lo detecta / líder técnico |
| 2. Contención | Aislar el servicio o la cuenta afectada y revocar credenciales comprometidas | Líder técnico |
| 3. Notificación | Avisar al equipo, a la institución y, si hay datos personales afectados, a la autoridad y a los titulares dentro de los plazos legales | Coordinador del proyecto |
| 4. Recuperación | Restaurar desde respaldos limpios y verificar la integridad | Equipo técnico |
| 5. Lecciones aprendidas | Documentar causa, impacto y mejoras; actualizar controles | Todo el equipo |

> ℹ️ Los roles y plazos concretos deben acordarse con el Equipo 4 y con la institución responsable del proyecto.

---

## 5. Riesgos identificados

Cada riesgo se relaciona con los controles que lo mitigan, de modo que se vea la trazabilidad entre riesgos y controles.

| # | Riesgo | Probabilidad | Impacto | Mitigación | Controles relacionados |
|---|--------|:------------:|:-------:|------------|------------------------|
| 1 | Credenciales subidas al repositorio | 🟠 Media | 🔴 Alto | `.gitignore` y revisión antes de cada commit | 4.2, 4.9 |
| 2 | Base de datos expuesta | 🟠 Media | 🔴 Alto | Restringir red y accesos; deshabilitar consola H2 | 4.4, 4.5 |
| 3 | Pérdida de datos | 🟠 Media | 🟠 Medio | Respaldos periódicos con prueba de restauración | 4.3, 4.10 |
| 4 | Dependencias desactualizadas | 🔴 Alta | 🟠 Medio | Actualizar y revisar librerías con regularidad | 4.6, 4.9 |
| 5 | Inyección SQL y XSS en el backend | 🟠 Media | 🔴 Alto | Validación de entradas, consultas parametrizadas y cabeceras de seguridad | 4.8 |
| 6 | Compromiso de cuentas administrativas (phishing) | 🟠 Media | 🔴 Alto | MFA, mínimo privilegio y cuentas administrativas separadas | 4.1 |
| 7 | Ransomware o borrado malicioso de respaldos | 🟢 Baja | 🔴 Alto | Copias en otra zona con acceso restringido | 4.10 |
| 8 | Amenaza interna (permisos excesivos) | 🟢 Baja | 🟠 Medio | Revisión periódica de permisos y registro de auditoría | 4.1, 4.7 |
| 9 | Incumplimiento de la LOPDP | 🟠 Media | 🔴 Alto | Inventario de datos, base legal, derechos de titulares y notificación de brechas | 4.7 |
| 10 | Caída o indisponibilidad del proveedor | 🟢 Baja | 🟠 Medio | Definir RPO/RTO y plan de recuperación | 4.10 |
| 11 | Dependencia de un solo proveedor (vendor lock-in) | 🟠 Media | 🟠 Medio | Usar tecnologías estándar y portables (por ejemplo, PostgreSQL, contenedores) | Recomendaciones 1 y 11 |
| 12 | Denegación de servicio sobre la API | 🟠 Media | 🟠 Medio | Rate limiting, WAF y protección DDoS | 4.5, 4.8 |

---

## 6. Criterio de priorización

La prioridad de cada control se asignó según el riesgo que reduce y el momento en que debe estar activo:

- **Alta:** el control mitiga riesgos de impacto alto, o su ausencia expondría datos personales o credenciales desde el primer día. Debe estar implementado **antes de cualquier despliegue**.
- **Media:** el control mejora la resiliencia y la capacidad de detección o recuperación. Puede implementarse **durante la operación**, pero con fecha comprometida.

---

## 7. Recomendaciones para el Equipo 4

1. **Base de datos de producción:** Descartar H2 para producción y migrar a un motor relacional en modalidad PaaS (ej. PostgreSQL o MySQL) gestionado por el proveedor, delegando así los respaldos, parches y reduciendo la carga operativa.
2. **Secretos:** Definir desde el inicio dónde se guardarán de forma segura (variables de entorno o gestor de secretos del proveedor).
3. **Red e Infraestructura:** Exponer el backend únicamente mediante HTTPS, limitar los puertos de red abiertos a lo estrictamente necesario y administrar por VPN o servidor bastión.
4. **Entornos:** Mantener una estricta separación física o lógica entre desarrollo, pruebas y producción.
5. **Accesos:** Documentar formalmente qué roles tienen permisos de administración y realizar revisiones periódicas.
6. **Cumplimiento legal (LOPDP):** Seleccionar infraestructura que facilite el cumplimiento de la normativa ecuatoriana de protección de datos (garantizando el derecho al olvido y el resguardo seguro) y verificar la región donde se almacenan los datos.
7. **DevSecOps:** Integrar herramientas de análisis estático de código (SAST) en GitLab para bloquear automáticamente commits que contengan credenciales o vulnerabilidades conocidas.
8. **APIs seguras:** Exigir tokens JWT de corta duración y aplicar limitación de tasa (*Rate Limiting*) para mitigar posibles ataques de denegación de servicio.
9. **Plan de incidentes:** Establecer un protocolo rápido de notificación y aislamiento del servicio en caso de detectar una brecha de seguridad (ver 4.11).
10. **Continuidad:** Definir RPO y RTO desde el diseño y guardar los respaldos en una zona distinta a la de producción.
11. **Portabilidad:** Preferir tecnologías estándar para reducir la dependencia de un único proveedor.

---

## 8. 🗓️ Hoja de ruta de implementación

| Etapa | Qué se implementa | Controles |
|-------|-------------------|-----------|
| **Antes del despliegue** | Todos los controles de prioridad Alta: accesos y MFA, secretos fuera del repositorio, HTTPS, cifrado en reposo, consola H2 deshabilitada, base de datos no expuesta, firewall, validación de entradas, revisión de código, inventario de datos personales y base legal | 4.1 a 4.9 (Alta) |
| **Primer mes de operación** | Registros y alertas, respaldos con primera prueba de restauración, separación de entornos, rate limiting, CORS y cabeceras, definición de RPO/RTO | 4.6, 4.8, 4.10 |
| **Mediano plazo (3 a 6 meses)** | Escaneo de dependencias y SAST en el pipeline, segmentación de red, WAF, rotación de credenciales, revisión de permisos, simulacro de incidente | 4.1, 4.2, 4.5, 4.9, 4.11 |
| **Continuo** | Parches, actualización de dependencias, revisión periódica de riesgos y controles | 4.5, 4.6 |

---

## 9. 📊 Indicadores de seguimiento

| Indicador | Meta sugerida |
|-----------|---------------|
| Cuentas administrativas con MFA | 100 % |
| Secretos detectados en el repositorio | 0 |
| Tiempo entre la publicación de un parche crítico y su aplicación | Menor a 7 días |
| Última prueba de restauración de respaldos exitosa | Dentro del último periodo acordado |
| Dependencias con vulnerabilidades críticas conocidas | 0 |
| Revisión de permisos de administración | Al menos cada 6 meses |
| Solicitudes de titulares atendidas dentro del plazo legal | 100 % |

---

## 10. 📝 Supuestos y limitaciones

- El análisis depende de lo definido por los Equipos 1, 2 y 4, en especial si el backend y la base de datos terminan en PaaS o IaaS.
- Las metas de los indicadores y los plazos son sugerencias y deben validarse con la institución responsable del proyecto.
- Los aspectos legales se presentan como guía técnica y deben ser revisados por la persona o área jurídica competente frente al texto vigente de la LOPDP y su reglamento.
- Los controles se asumen para un proyecto académico con intención de pasar a un entorno real; si permanece solo como prototipo, algunos pueden reducirse.

---

## 11. 📖 Glosario

| Término | Significado |
|---------|-------------|
| **IaaS** | Infraestructura como servicio: el proveedor ofrece servidores y red; el equipo administra el sistema operativo y lo que corre encima |
| **PaaS** | Plataforma como servicio: el proveedor administra también el sistema operativo y la plataforma de ejecución |
| **SaaS** | Software como servicio: el proveedor entrega la aplicación completa |
| **MFA** | Autenticación multifactor: verificación con más de un factor (por ejemplo, contraseña y código) |
| **TLS / HTTPS** | Protocolo que cifra la comunicación entre el usuario y el servidor |
| **SAST** | Análisis estático de seguridad del código fuente |
| **JWT** | Token firmado que se usa para autenticar peticiones a una API |
| **WAF** | Firewall de aplicaciones web |
| **RPO / RTO** | Pérdida máxima de datos tolerable / tiempo máximo de caída tolerable |
| **LOPDP** | Ley Orgánica de Protección de Datos Personales del Ecuador |

---

## 12. 📚 Referencias sugeridas

- Modelo de responsabilidad compartida del proveedor cloud elegido.
- Ley Orgánica de Protección de Datos Personales (LOPDP) del Ecuador y su reglamento.
- OWASP Top 10 y OWASP Application Security Verification Standard (ASVS).
- CIS Controls y la norma ISO/IEC 27001 como marco de buenas prácticas.
- Documentos `nube-H1.md` y `nube-H2.md` de los Equipos 1 y 2.

---

## Conclusión

La seguridad del proyecto es **compartida**: el proveedor cubre la infraestructura base y nuestro equipo es responsable de la aplicación, los datos, los accesos y la configuración. Aplicar los controles descritos en este documento, empezando por los de prioridad Alta antes del despliegue, reduce de forma significativa los riesgos más críticos del sistema y deja al Equipo 4 una base clara para decidir la arquitectura.
