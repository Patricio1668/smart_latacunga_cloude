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

El despliegue en la nube pública no es la opción por defecto: cada componente se ubica según sus requisitos de seguridad, latencia, privacidad, dependencia de hardware y costo.

### 4.1 Criterios de decisión

| Criterio | Pregunta de evaluación | Si la respuesta es "sí" |
|---|---|---|
| Secretos | ¿Maneja credenciales, claves privadas o tokens? | Lab (almacenamiento seguro local) |
| Datos sensibles | ¿Procesa o almacena datos personales o confidenciales? | Lab / entorno controlado |
| Hardware | ¿Necesita acceso directo a sensores, dispositivos o controladores? | Edge / lab |
| Latencia | ¿Requiere respuesta en tiempo real o muy baja latencia? | Edge |
| Conectividad | ¿Debe operar aunque se caiga Internet? | Edge / lab |
| Respaldos y registros | ¿Genera copias o logs confidenciales? | Almacenamiento protegido local |
| Costo | ¿Mueve grandes volúmenes de datos o requiere recursos siempre activos? | Evaluar local o híbrido |

### 4.2 Ubicación por tipo de componente

| Tipo de componente | Lab | Edge | Nube pública |
|---|---|---|---|
| Credenciales, claves y secretos | ✔ | ✘ | ✘ |
| Datos sensibles de usuarios | ✔ | ✘ | Solo cifrados y anonimizados |
| Sensores y dispositivos físicos | ✘ | ✔ | ✘ |
| Servicios de baja latencia | ✔ | ✔ | ✘ |
| Sistemas dependientes de hardware | ✔ | ✔ | ✘ |
| Copias de seguridad y registros confidenciales | ✔ | ✘ | Solo cifrados |
| Paneles de visualización y documentación | ✔ | ✘ | ✔ |
| Análisis posterior con datos anonimizados | ✔ | ✘ | ✔ |
| Repositorio de código (sin secretos) | ✔ | ✘ | ✔ |

### 4.3 Condiciones para evaluar la nube pública

Un componente solo puede desplegarse en la nube si cumple **todas** estas condiciones:

| Condición | Detalle |
|---|---|
| Independencia del hardware | No depende de dispositivos ni de la red local. |
| Sin información crítica | No maneja secretos ni datos sensibles, o estos van cifrados y con acceso restringido. |
| Tolerancia a la latencia | Funciona correctamente con la comunicación remota. |
| Tolerancia a desconexión | Una caída de Internet no afecta la operación crítica. |
| Costo aceptable | El gasto de ejecución y transferencia de datos es razonable. |

### 4.4 Despliegue híbrido

Cuando un componente cumple solo parcialmente los criterios, se divide en dos partes:

| Parte | Ubicación | Ejemplos |
|---|---|---|
| Crítica | Lab / edge | Captura de datos, control de dispositivos, manejo de secretos. |
| No crítica | Nube pública | Visualización, documentación, análisis con datos anonimizados. |

### 4.5 Proceso de decisión

| Paso | Acción | Resultado |
|---|---|---|
| 1 | Evaluar el componente con la tabla 4.1. | Si cumple algún criterio, permanece en lab o edge. |
| 2 | Si no cumple ninguno, evaluarlo con 4.3. | Si cumple todas las condiciones, es candidato a la nube. |
| 3 | Si el resultado es parcial, aplicar 4.4. | Se divide en parte crítica y no crítica. |
| 4 | Documentar la decisión. | Queda registrada la ubicación y su justificación. |

## 5. Medidas de protección

- Aplicar permisos de acceso según las funciones de cada usuario.
- No subir contraseñas, tokens ni claves privadas al repositorio.
- Proteger los datos sensibles mediante controles de acceso y cifrado.
- Mantener copias de seguridad protegidas y verificar su recuperación.
- Documentar los componentes excluidos y justificar cada decisión.

## 6. Conclusión

No todos los componentes de un sistema deben desplegarse en la nube pública. Mantener determinados recursos en el laboratorio o en el edge permite responder a requisitos de latencia, seguridad, privacidad y dependencia del hardware. La arquitectura debe definir explícitamente qué servicios se despliegan en cada entorno y por qué.
