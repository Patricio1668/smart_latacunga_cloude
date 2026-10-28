H3 — Límites del despliegue en la nube pública
1. Objetivo

Identificar los componentes que deben permanecer en el laboratorio local (lab) o en el borde de la red (edge), evitando su despliegue en la nube pública cuando existan requisitos de seguridad, latencia, privacidad, disponibilidad, compatibilidad o costo que lo justifiquen.

El objetivo es establecer criterios claros para decidir qué componentes deben ejecutarse localmente, cuáles pueden desplegarse en la nube pública y cuáles requieren una arquitectura híbrida.

2. Lista de exclusiones
Componente	Ubicación propuesta	Justificación
Credenciales, claves privadas y secretos	Lab / almacén seguro	Reducir el riesgo de exposición y accesos no autorizados mediante una gestión segura de secretos.
Datos sensibles de usuarios	Lab / entorno controlado	Minimizar los riesgos de filtración y garantizar el cumplimiento de los requisitos de privacidad aplicables.
Dispositivos físicos y sensores locales	Edge / red local	Necesitan conectividad directa con los equipos y, en muchos casos, una respuesta rápida.
Servicios que requieren baja latencia	Edge / lab	Evitar retrasos asociados a la comunicación con servidores remotos.
Sistemas dependientes de hardware específico	Lab / edge	Pueden necesitar dispositivos, controladores o configuraciones que no están disponibles en la nube.
Copias de seguridad y registros confidenciales	Almacenamiento protegido	Garantizar la confidencialidad, integridad y recuperación de la información. Los respaldos externos cifrados pueden utilizarse cuando estén autorizados y adecuadamente protegidos.
3. Riesgos de desplegar componentes en la nube pública
Seguridad: una configuración incorrecta, una vulnerabilidad o la exposición de credenciales puede facilitar accesos no autorizados.
Privacidad: los datos personales o confidenciales pueden quedar expuestos si no existen controles de acceso, cifrado y medidas de protección adecuadas.
Latencia: la comunicación con servidores remotos puede retrasar las operaciones que requieren respuestas inmediatas.
Disponibilidad: una interrupción de Internet o del proveedor puede impedir el acceso a servicios que dependan completamente de la nube.
Costos: la transferencia de grandes volúmenes de datos, el almacenamiento y el uso continuo de recursos pueden incrementar los gastos.
Compatibilidad: algunos componentes pueden depender de hardware, controladores o servicios disponibles únicamente en el entorno local.
Cumplimiento normativo: el almacenamiento o procesamiento de información puede estar sujeto a obligaciones legales, contractuales o institucionales.
4. Límites del despliegue

El despliegue en la nube pública no debe considerarse la opción predeterminada para todos los componentes. Cada uno debe ubicarse según sus requisitos de seguridad, privacidad, latencia, dependencia del hardware, continuidad operativa y costo.

La decisión debe basarse en una evaluación de riesgos y en los requisitos del proyecto, diferenciando las restricciones obligatorias de los riesgos que pueden mitigarse mediante controles técnicos y organizativos.

4.1 Criterios de decisión
Criterio	Pregunta de evaluación	Decisión orientativa
Secretos	¿Maneja credenciales, claves privadas o tokens?	Utilizar un gestor de secretos seguro y restringir su exposición. Mantenerlos localmente cuando así lo exijan los requisitos del proyecto.
Datos sensibles	¿Procesa o almacena datos personales o confidenciales?	Priorizar un entorno controlado y evaluar las condiciones de protección antes de autorizar la nube.
Hardware	¿Necesita acceso directo a sensores, dispositivos o controladores?	Edge / lab.
Latencia	¿Requiere respuestas en tiempo real o muy baja latencia?	Edge / lab, según los requisitos temporales.
Conectividad	¿Debe operar aunque se interrumpa Internet?	Mantener localmente las funciones críticas y habilitar mecanismos de funcionamiento sin conexión.
Respaldos y registros	¿Genera copias o registros confidenciales?	Aplicar cifrado, controles de acceso, políticas de retención y pruebas de recuperación.
Costo	¿Transfiere grandes volúmenes de datos o requiere recursos siempre activos?	Comparar los costos de las alternativas local, nube e híbrida.
Cumplimiento	¿Existen requisitos legales, contractuales o institucionales sobre los datos?	Verificar su cumplimiento antes de autorizar el despliegue.

Estos criterios sirven para identificar riesgos y restricciones. El cumplimiento de uno de ellos no implica automáticamente que todo el componente deba permanecer en el entorno local; la decisión depende de la criticidad, los controles disponibles y los requisitos aplicables.

4.2 Ubicación por tipo de componente
Tipo de componente	Lab	Edge	Nube pública
Credenciales, claves y secretos	✔ Almacén seguro	✔ Cuando sea necesario y con protección adecuada	✔ Solo mediante una solución de gestión de secretos autorizada y correctamente configurada
Datos sensibles de usuarios	✔ Entorno controlado	✔ Si es necesario para la operación local	✔ Solo tras evaluar los riesgos, la protección y los requisitos legales aplicables
Sensores y dispositivos físicos	✘ No suele ser su ubicación física	✔ Ubicación preferente	✘ No sustituye el acceso físico directo
Servicios de baja latencia	✔ Si cumple los requisitos temporales	✔ Ubicación preferente	Solo si se demuestra que cumple los requisitos de latencia
Sistemas dependientes de hardware	✔ Si dispone del hardware necesario	✔ Si dispone del hardware necesario	Solo si la infraestructura ofrece la compatibilidad requerida
Copias de seguridad y registros confidenciales	✔ Con protección adecuada	✔ Cuando sea necesario	✔ Si están cifrados, tienen acceso restringido y su almacenamiento está autorizado
Paneles de visualización y documentación	✔	✔ Si se necesita acceso local	✔ Si no exponen información confidencial y se protegen adecuadamente
Análisis posterior con datos anonimizados	✔	✔ Si requiere procesamiento local	✔ Cuando los datos y el tratamiento cumplen los requisitos aplicables
Repositorio de código sin secretos	✔	✘ No suele ser necesario	✔ En un servicio autorizado con controles de acceso

Nota: la ubicación propuesta es orientativa. La decisión final debe considerar la arquitectura, la sensibilidad de la información, los controles de seguridad y los requisitos operativos.

4.3 Condiciones para evaluar el despliegue en la nube pública

Un componente podrá desplegarse en la nube pública cuando una evaluación técnica y de seguridad determine que su funcionamiento es compatible con los requisitos del proyecto y que los riesgos identificados se encuentran adecuadamente mitigados.

Condición	Detalle
Independencia del hardware	El componente no requiere acceso directo a dispositivos físicos locales, o cuenta con una arquitectura que resuelve esa dependencia.
Protección de la información	Los secretos y datos sensibles cuentan con controles adecuados de acceso, cifrado y gestión de claves.
Tolerancia a la latencia	El componente funciona correctamente con los tiempos de comunicación previstos.
Continuidad operativa	Una interrupción de Internet no compromete las funciones críticas del sistema.
Costo aceptable	Los gastos de procesamiento, almacenamiento, transferencia y mantenimiento son compatibles con el presupuesto del proyecto.
Cumplimiento y seguridad	El despliegue satisface las políticas de seguridad y las obligaciones legales, contractuales e institucionales aplicables.
Recuperación ante fallos	Existen mecanismos adecuados de respaldo, recuperación y continuidad del servicio según su criticidad.

Si alguna condición no se cumple, se deben definir controles adicionales, mantener las funciones afectadas en el entorno local o evaluar una arquitectura híbrida. Los requisitos legales o de seguridad obligatorios no pueden sustituirse simplemente por una aceptación del riesgo.

4.4 Despliegue híbrido

Cuando un componente combina funciones críticas locales con funciones que pueden ejecutarse en la nube, se puede dividir su arquitectura para asignar cada función al entorno más adecuado.

Parte	Ubicación	Ejemplos
Crítica	Lab / edge	Captura de datos, control de dispositivos, procesamiento en tiempo real y gestión local de secretos.
No crítica	Nube pública, cuando esté autorizada	Visualización, documentación, análisis posterior con datos debidamente protegidos y procesamiento de información anonimizada.
Comunicación entre entornos	Canal protegido	Transmisión autenticada y cifrada de los datos estrictamente necesarios entre el entorno local y la nube.

La arquitectura híbrida debe aplicar autenticación, autorización, cifrado, validación de datos y controles de acceso en las comunicaciones entre los entornos.

4.5 Proceso de decisión de despliegue
Paso	Acción	Resultado
1	Evaluar los requisitos de seguridad, privacidad, hardware, latencia, conectividad, cumplimiento y costo.	Se identifican las restricciones y los riesgos del componente.
2	Determinar si existen requisitos obligatorios que impidan su despliegue en la nube.	Se establece si debe permanecer total o parcialmente en el entorno local.
3	Evaluar las condiciones de la sección 4.3 para los componentes candidatos a la nube.	Se determina si el despliegue es viable y qué controles necesita.
4	Analizar alternativas híbridas cuando existan funciones críticas y no críticas.	Se separan las funciones locales de las que pueden ejecutarse remotamente.
5	Documentar la ubicación, la justificación, los riesgos y los controles aplicados.	Se obtiene un registro verificable de la decisión.
6	Revisar la decisión ante cambios importantes en la arquitectura, los requisitos o las amenazas.	Se mantiene actualizada la estrategia de despliegue.
5. Medidas de protección

Las medidas de protección reducen los riesgos asociados al almacenamiento, procesamiento y transmisión de información. Deben aplicarse según el tipo de componente, su criticidad y el entorno donde se despliegue.

5.1 Medidas por área
Área	Medida	Objetivo
Control de acceso	Asignar permisos según las funciones de cada usuario y aplicar el principio de mínimo privilegio.	Evitar accesos innecesarios o no autorizados.
Secretos	No subir contraseñas, tokens ni claves privadas al repositorio. Utilizar un gestor de secretos apropiado para el entorno.	Evitar la exposición de credenciales.
Datos sensibles	Aplicar cifrado en reposo y en tránsito, controles de acceso y minimización de datos.	Reducir el riesgo de filtraciones y accesos indebidos.
Copias de seguridad	Mantener respaldos protegidos, cifrados y separados cuando sea necesario. Verificar periódicamente su recuperación.	Garantizar la recuperación de la información.
Registros (logs)	Evitar incluir contraseñas, tokens y datos personales innecesarios. Restringir el acceso y establecer períodos de retención.	Prevenir filtraciones a través de los registros.
Red	Limitar los puertos y servicios expuestos; separar las redes del lab y del edge de las redes públicas.	Reducir la superficie de ataque.
Comunicación	Utilizar conexiones autenticadas y cifradas entre los componentes locales y remotos.	Proteger los datos durante su transmisión.
Actualizaciones	Aplicar actualizaciones de seguridad y revisar las dependencias de software.	Reducir la exposición a vulnerabilidades conocidas.
Documentación	Registrar los componentes excluidos de la nube y justificar cada decisión.	Mantener la trazabilidad y facilitar las auditorías.
5.2 Buenas prácticas para el repositorio
Práctica	Detalle
Archivo .gitignore	Incluir archivos de configuración local, archivos .env, claves privadas, certificados y otros archivos sensibles que no deban versionarse.
Archivo de ejemplo	Utilizar un archivo .env.example con nombres de variables y valores ficticios, sin credenciales reales.
Revisión antes de subir	Comprobar que los cambios no incluyan contraseñas, tokens, claves privadas ni datos sensibles.
Detección de secretos	Utilizar herramientas de detección de secretos en el repositorio y, cuando sea posible, en la integración continua.
Rotación de credenciales	Si una credencial queda expuesta, revocarla o cambiarla de inmediato. Eliminarla del repositorio no es suficiente.
Protección del repositorio	Configurar los permisos de acceso y las protecciones de ramas según las necesidades del proyecto.

El archivo .gitignore ayuda a evitar que se agreguen archivos sensibles por accidente, pero no elimina los secretos que ya están registrados en el historial del repositorio.

5.3 Verificación periódica
Verificación	Frecuencia sugerida	Responsable
Revisar los permisos de usuarios	Mensual y ante cambios de personal o funciones	Administrador del lab / responsable de seguridad
Probar la recuperación de copias de seguridad	Trimestral o según la criticidad del sistema	Administrador del lab
Revisar la presencia de secretos en el repositorio	En cada cambio importante y mediante controles automatizados	Equipo de desarrollo
Revisar las dependencias y actualizaciones de seguridad	Periódicamente y ante alertas relevantes	Equipo de desarrollo
Actualizar la lista de componentes excluidos	Cuando cambie la arquitectura o el nivel de riesgo	Equipo del proyecto
Revisar los controles de acceso y las políticas de retención	Periódicamente y ante cambios relevantes	Responsable de seguridad / administrador

Las frecuencias indicadas son orientativas y deben ajustarse al nivel de riesgo, los requisitos del proyecto y las políticas institucionales.

6. Conclusión

No todos los componentes de un sistema deben desplegarse en la nube pública. La decisión debe basarse en las características de cada componente y en criterios definidos, en lugar de depender únicamente de preferencias o comodidad.

Aspecto	Conclusión
Seguridad y privacidad	Las credenciales, claves y datos sensibles requieren controles adecuados de protección. Su permanencia en el entorno local dependerá de los requisitos de seguridad, privacidad y cumplimiento aplicables.
Latencia y hardware	Los sensores, dispositivos físicos y servicios de respuesta inmediata suelen necesitar procesamiento local o en el edge.
Disponibilidad	Las funciones críticas deben poder continuar operando ante interrupciones de Internet cuando así lo exijan los requisitos del sistema.
Uso de la nube	La nube pública puede utilizarse para componentes compatibles con los requisitos de seguridad, latencia, disponibilidad, costo y cumplimiento.
Despliegue híbrido	Las funciones críticas pueden mantenerse en el lab o edge, mientras que las funciones no críticas pueden ejecutarse en la nube si están adecuadamente protegidas.
Protección	El control de acceso, el cifrado, la gestión de secretos, las actualizaciones y las copias de seguridad verificadas son necesarios en todos los entornos.
Trazabilidad	Las decisiones de despliegue deben documentarse y revisarse cuando cambien la arquitectura, los requisitos o las amenazas.
6.1 Cierre

La arquitectura debe definir explícitamente qué servicios se despliegan en cada entorno (lab, edge o nube pública) y justificar cada decisión mediante el proceso descrito en la sección 4.5.

Esta estrategia permite equilibrar seguridad, privacidad, rendimiento, disponibilidad y costo, al tiempo que facilita la revisión y mejora de la arquitectura a medida que evolucionan los requisitos y las amenazas del proyecto.
