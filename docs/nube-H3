# H3 — Límites del despliegue en la nube pública

## 1. Objetivo

Identificar los componentes que deben permanecer en el laboratorio local (lab) o en el borde de la red (edge), evitando su despliegue en la nube pública por razones de seguridad, latencia, privacidad y disponibilidad.

## 2. Lista de exclusiones

| Componente | Ubicación propuesta | Justificación |
|---|---|---|
| Credenciales, claves privadas y secretos | Lab / almacenamiento seguro local | Evitar la exposición de credenciales y accesos no autorizados. |
| Datos sensibles de usuarios | Lab / entorno controlado | Reducir el riesgo de filtraciones y accesos indebidos. |
| Dispositivos físicos y sensores locales | Edge / red local | Necesitan conectividad directa con los equipos y una respuesta rápida. |
| Servicios que requieren baja latencia | Edge / lab | Evitar retrasos ocasionados por la comunicación con servidores remotos. |
| Sistemas dependientes de hardware específico | Lab | Pueden requerir dispositivos, controladores o configuraciones que no están disponibles en la nube. |
| Copias de seguridad y registros confidenciales | Almacenamiento protegido | Evitar que información sensible quede expuesta o sin controles adecuados. |

## 3. Riesgos de desplegar estos componentes en la nube pública

- **Seguridad:** una configuración incorrecta o credenciales expuestas pueden facilitar accesos no autorizados.
- **Privacidad:** los datos sensibles pueden quedar expuestos si no existen controles de acceso y protección adecuados.
- **Latencia:** la comunicación con servidores remotos puede retrasar las operaciones que requieren respuestas inmediatas.
- **Disponibilidad:** una interrupción de Internet puede impedir el acceso a servicios que dependan completamente de la nube.
- **Costos:** transferir grandes volúmenes de datos o mantener recursos activos puede incrementar los gastos.
- **Compatibilidad:** algunos componentes pueden depender de hardware o servicios disponibles únicamente en el entorno local.

## 4. Límites del despliegue

Los componentes que requieran acceso directo al hardware, baja latencia o controles especiales sobre datos sensibles permanecerán en el laboratorio local o en el entorno edge, según sus necesidades.

Los servicios que puedan ejecutarse de forma segura y que no dependan de recursos físicos locales podrán evaluarse para su despliegue en la nube pública.

La decisión final debe considerar los requisitos de seguridad, disponibilidad, rendimiento, privacidad y costo de cada componente.

## 5. Medidas de protección

- Aplicar permisos de acceso según las funciones de cada usuario.
- No subir contraseñas, tokens ni claves privadas al repositorio.
- Proteger los datos sensibles mediante controles de acceso y cifrado.
- Mantener copias de seguridad protegidas y verificar su recuperación.
- Documentar los componentes excluidos y justificar cada decisión.

## 6. Conclusión

No todos los componentes de un sistema deben desplegarse en la nube pública. Mantener determinados recursos en el laboratorio o en el edge permite responder a requisitos de latencia, seguridad, privacidad y dependencia del hardware. La arquitectura debe definir explícitamente qué servicios se despliegan en cada entorno y por qué.
