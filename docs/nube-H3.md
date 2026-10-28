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

### 4.4  Despliegue híbrido
Cuando un componente cumple solo parcialmente los criterios, se divide en dos partes:

| Parte | Ubicación | Ejemplos |
|---|---|---|
| Crítica | Lab / edge | Captura de datos, control de dispositivos, manejo de secretos. |
| No crítica | Nube pública | Visualización, documentación, análisis con datos anonimizados. |

### 4.5 Proceso de Edición

| Paso | Acción | Resultado |
|---|---|---|
| 1 | Evaluar el componente con la tabla 4.1. | Si cumple algún criterio, permanece en lab o edge. |
| 2 | Si no cumple ninguno, evaluarlo con 4.3. | Si cumple todas las condiciones, es candidato a la nube. |
| 3 | Si el resultado es parcial, aplicar 4.4. | Se divide en parte crítica y no crítica. |
| 4 | Documentar la decisión. | Queda registrada la ubicación y su justificación. |

## 5. Medidas de protección

Las medidas de protección reducen los riesgos de mantener o desplegar cada componente. Se aplican según el tipo de información o recurso que se maneja.

### 5.1 Medidas por área

| Área | Medida | Objetivo |
|---|---|---|
| Control de acceso | Asignar permisos según las funciones de cada usuario (principio de mínimo privilegio). | Evitar accesos innecesarios o no autorizados. |
| Secretos | No subir contraseñas, tokens ni claves privadas al repositorio; usar variables de entorno o un almacén seguro local. | Evitar la exposición de credenciales. |
| Datos sensibles | Proteger con cifrado (en reposo y en tránsito) y controles de acceso. | Reducir el riesgo de filtraciones. |
| Copias de seguridad | Mantenerlas protegidas, cifradas y verificar periódicamente su recuperación. | Garantizar la recuperación sin exponer información. |
| Registros (logs) | Evitar guardar datos sensibles en los registros y restringir quién puede consultarlos. | Evitar filtraciones por logs. |
| Red | Limitar los puertos y servicios expuestos; separar la red del lab y del edge de la red pública. | Reducir la superficie de ataque. |
| Documentación | Registrar los componentes excluidos de la nube y justificar cada decisión. | Mantener trazabilidad y claridad. |

### 5.2 Buenas prácticas para el repositorio

| Práctica | Detalle |
|---|---|
| Archivo .gitignore | Incluir archivos de configuración local, claves, certificados y archivos .env. |
| Archivo de ejemplo | Usar un archivo .env.example con nombres de variables sin valores reales. |
| Revisión antes de subir | Verificar que ningún commit contenga credenciales o datos sensibles. |
| Rotación de credenciales | Si una credencial se expone, cambiarla de inmediato, aunque se haya borrado del repositorio. |

### 5.3 Verificación periódica

| Verificación | Frecuencia sugerida | Responsable |
|---|---|---|
| Revisar permisos de usuarios | Mensual | Administrador del lab |
| Probar la recuperación de copias de seguridad | Trimestral | Administrador del lab |
| Revisar que no haya secretos en el repositorio | En cada cambio importante | Equipo de desarrollo |
| Actualizar la lista de componentes excluidos | Cuando cambie la arquitectura | Equipo del proyecto |

## 6. Conclusión

No todos los componentes de un sistema deben desplegarse en la nube pública. La decisión depende de las características de cada uno y debe tomarse con criterios definidos, no por preferencia o comodidad.

| Aspecto | Conclusión |
|---|---|
| Seguridad y privacidad | Las credenciales, claves y datos sensibles permanecen en el lab o en un entorno controlado, para reducir el riesgo de exposición. |
| Latencia y hardware | Los sensores, dispositivos físicos y servicios de respuesta inmediata se mantienen en el edge o en el lab, por su dependencia de conectividad directa y hardware específico. |
| Disponibilidad | Los servicios críticos deben seguir operando aunque se pierda la conexión a Internet. |
| Uso de la nube | La nube pública se reserva para componentes que no dependen del hardware local, toleran la latencia y no manejan información crítica, o la manejan cifrada y anonimizada. |
| Despliegue híbrido | Cuando un componente cumple los criterios solo en parte, se divide: la parte crítica queda en el lab o edge y la no crítica puede ir a la nube. |
| Protección | Los controles de acceso, el cifrado, la gestión de secretos y las copias de seguridad verificadas son necesarios sin importar dónde se despliegue cada componente. |

### 6.1 Cierre

La arquitectura debe definir de forma explícita qué servicios se despliegan en cada entorno (lab, edge o nube pública) y por qué, aplicando el proceso de decisión de la sección 4 y documentando cada resultado. Así se logra un equilibrio entre seguridad, privacidad, rendimiento, disponibilidad y costo, y se facilita la revisión y mejora de la arquitectura a medida que el proyecto evolucione.
