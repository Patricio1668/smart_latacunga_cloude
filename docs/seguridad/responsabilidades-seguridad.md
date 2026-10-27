#  Responsabilidades y seguridad

**Proyecto:** Nube inteligente de Latacunga
**Equipo 3** · Responsabilidades y seguridad

> 📥 Parte de: Equipo 1 (inventario de servicios) y Equipo 2 (clasificación cloud)
> 📤 Alimenta a: Equipo 4 (decisiones de arquitectura)

---

## 📌 Resumen ejecutivo

El presente documento establece la distribución de responsabilidades entre el proveedor de servicios en la nube y el equipo del proyecto **Nube inteligente de Latacunga**, conforme al modelo de responsabilidad compartida, con el fin de garantizar que cada componente de la solución cuente con un responsable claramente definido.

Su alcance comprende los cuatro componentes principales del sistema: **backend, base de datos, infraestructura y pruebas**. Para cada uno se determina qué aspectos de seguridad son cubiertos por el proveedor y cuáles corresponden al equipo, tales como la gestión de accesos, la protección de credenciales, la configuración de la base de datos, los respaldos y la actualización de dependencias.

Adicionalmente, se identifican los riesgos más relevantes, se definen los controles de seguridad mínimos con su nivel de prioridad y se formulan recomendaciones técnicas dirigidas al **Equipo 4**, de modo que sirvan como insumo para la toma de decisiones de arquitectura y para la posterior integración en el documento `nube-H2.md`.

---

## 1. 🎯 Objetivo

- Aclarar las responsabilidades del proveedor cloud y del equipo.
- Identificar los riesgos de seguridad principales.
- Proponer controles claros para reducirlos.
- Dar recomendaciones concretas al Equipo 4 para sus decisiones de arquitectura.

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

---

## ✅ Conclusión

La seguridad del proyecto es **compartida**: el proveedor cubre la infraestructura base y nuestro equipo es responsable de la aplicación, los datos, los accesos y la configuración. Aplicar los controles de este documento reduce los riesgos más críticos.
