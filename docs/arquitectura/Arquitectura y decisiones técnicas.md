*AgriSmart Latacunga · H2 · Equipo 4*  *UTC · Infraestructura, Seguridad y Gestión de Riesgo en las TI*

## 1. Contexto y restricciones

AgriSmart Latacunga evalúa el impacto de una app móvil con IA en la calidad del suelo y la productividad agrícola. El piloto usa 40 nodos ESP32 con LoRa, un nodo fog en los servidores HP de la UTC, una API en AWS y una app Flutter que funciona sin señal.

Las restricciones del proyecto son las que explican casi todas las decisiones:

| Restricción | Valor o condición | Efecto en el diseño |
| --- | --- | --- |
| Conectividad rural intermitente | Riesgo alto | El fog captura, guarda y alerta sin internet; la nube consolida, no es el único lugar de verdad |
| Presupuesto de nube | ≈ 83,33 USD/mes; 2.000 USD en 24 meses | Servicios administrados, una sola zona y nada de NAT ni API Gateway |
| Equipo pequeño y rotativo | 4 docentes y estudiantes de 5.º a 8.º | Menos operación; una sola unidad desplegable; sin microservicios |
| Datos personales | LOPDP y consentimiento | RDS privado, cifrado, secretos administrados y mínimo privilegio |
| Volumen bajo | 40 nodos × 96 lecturas/día ≈ 1,4 millones/año | PostgreSQL estándar es suficiente; no hace falta Kafka ni TimescaleDB en la nube |

**Dentro del piloto:** nodos ESP32, gateway LoRaWAN, fog en Docker Compose, API + BFF en Fargate, PostgreSQL en RDS, app Flutter offline-first, panel web, Cognito, Secrets Manager y observabilidad.

**Fuera del piloto:** Kafka, Kubernetes, multi-región, RDS Multi-AZ, API Gateway, WAF y microservicios. Se reevalúan si el proyecto escala.

## 2. Arquitectura en una vista

La arquitectura tiene cuatro zonas. Cada una hace una cosa distinta y no se mezclan responsabilidades:

| Zona | Qué hace | Dónde vive |
| --- | --- | --- |
| **Edge** | Mide pH, humedad, temperatura y NPK; envía por LoRaWAN | Parcela |
| **Fog** | Recibe, guarda, alerta y sincroniza; sigue funcionando sin internet | Servidores HP de la UTC o de la parroquia |
| **Cloud** | Consolida datos, publica la API y entrena modelos | AWS |
| **Clientes** | App Flutter y panel web | Teléfonos y navegadores |

**Figura 1.** *Vista de contenedores: edge → fog → AWS → clientes. El fog y la app sincronizan contra la API; la base de datos permanece en subred privada.*

### Recorrido de una lectura, en 4 pasos

1. El ESP32 mide cada 15 minutos y guarda la lectura con un identificador formado por el nodo y un número de secuencia.
2. ChirpStack recibe la lectura por LoRaWAN y la publica en MQTT; el servicio de ingesta la guarda en TimescaleDB y evalúa las alertas.
3. Cuando hay internet, el sincronizador envía los lotes a la API por HTTPS/mTLS. La API aplica idempotencia y escribe en RDS.
4. La app consulta la API y conserva sus datos en SQLite para seguir funcionando sin señal.

> **Regla de oro:** el fog nunca escribe directo en RDS. Todo pasa por la API. Eso protege la base de datos y centraliza las validaciones.

## 3. Decisión A: API y BFF en ECS Fargate

> **Decisión:** contenedores Docker en **Amazon ECS Fargate**. La misma imagen que el equipo ya usa en Docker Compose se despliega en AWS sin administrar servidores.

### Por qué Fargate

- **Menos operación:** AWS administra el servidor y el runtime; el equipo no parchea ni reinicia máquinas.
- **Portabilidad:** se conserva la imagen Docker y el docker-compose del fog; migrar a otro proveedor o a EC2 es posible.
- **Costo razonable:** ≈ 19,80 USD/mes con 0,5 vCPU y 1 GB, con un promedio de 1,1 tareas.
- **Encaje con el equipo:** una sola unidad desplegable para API + BFF; no se obliga a reescribir como funciones.

### Comparación de opciones de API

| Criterio | A1. Docker en EC2 | A2. ECS Fargate | A3. Lambda + API Gateway |
| --- | --- | --- | --- |
| Modelo de servicio | IaaS | PaaS de contenedores | FaaS |
| Parches del sistema operativo | Equipo | AWS | AWS |
| Portabilidad desde Docker Compose | Total | Alta | Baja: reescribir el BFF |
| Costo mensual aproximado | ≈ 15 USD + operación | ≈ 19,80 USD | ≈ 0–5 USD + RDS Proxy |
| Conexión a PostgreSQL | Pool estable | Pool estable | Requiere RDS Proxy |
| Operación para estudiantes | Alta | Media | Media; depuración más difícil |
| Veredicto | Plan B | **Elegida** | Descartada para el piloto |

### Qué se sacrifica en la API

Costo medio: Fargate tiene un costo mayor que una instancia EC2 pequeña, pero permite ahorrar tiempo y trabajo en su administración. Si el presupuesto es limitado, se puede utilizar la misma imagen en una instancia EC2 t3.small.

Arranque de tarea: Cuando se crea una nueva tarea, esta tarda algunas decenas de segundos en iniciarse, no milisegundos. Esto no representa un problema para el piloto, ya que el ALB mantiene una tarea activa y el autoscaling permite aumentar de 1 a 2 tareas.

Menos control del host: Fargate no permite acceder directamente al servidor donde se ejecuta la aplicación. Sin embargo, esto se puede compensar mediante el uso de logs, métricas y trazas con CloudWatch/OpenTelemetry.

**Costo de esta decisión:** ≈ **19,80 USD/mes** (0,5 vCPU + 1 GB, promedio 1,1 tareas). El ALB y la IP pública se detallan en la sección 6.

## 4. Decisión B: PostgreSQL en Amazon RDS

> **Decisión:** **PostgreSQL 16 en Amazon RDS**, instancia *db.t4g.micro*, 20 GB gp3, una sola zona, respaldos automáticos de 7 días y cifrado en reposo.

### Por qué RDS

- **El modelo es relacional:** agricultores, parcelas, cultivos, lecturas y recomendaciones tienen relaciones claras; SQL es la herramienta natural.
- **El estudio necesita análisis:** diferencias en diferencias (DID) y propensity score matching (PSM) se resuelven con SQL directo y exportación a Python/R.
- **AWS administra lo aburrido:** parches, backups y recuperación; el equipo se concentra en el esquema, los datos y la app.
- **Idempotencia sencilla:** una restricción única (device_id, seq) y ON CONFLICT DO NOTHING evitan duplicados.
- **Costo bajo y predecible:** ≈ 14 USD/mes para el volumen del piloto.

### Comparación de opciones de base de datos

| Criterio | B1. PostgreSQL en RDS | B2. PostgreSQL en EC2 | B3. DynamoDB |
| --- | --- | --- | --- |
| Modelo de servicio | PaaS | IaaS | PaaS NoSQL |
| Ajuste al modelo relacional | Alto | Alto | Bajo |
| Análisis SQL del estudio | Directo | Directo | Requiere exportar y transformar |
| Parches y respaldos | AWS | Equipo | AWS |
| Idempotencia | Restricción única + ON CONFLICT | Igual | Escritura condicional |
| Costo mensual aproximado | ≈ 14 USD | ≈ 17 USD + operación | ≈ 1–5 USD |
| Alta disponibilidad | Multi-AZ opcional | Hay que construirla | Incluida |
| Veredicto | **Elegida** | Plan B | Descartada |

### Qué se sacrifica en la base de datos

- **Una sola zona:** si falla la zona de AWS, la nube puede quedar fuera unas horas. El fog sigue capturando y la app usa SQLite; al volver la conexión, se sincroniza. Multi-AZ cuesta ≈ 15 USD/mes más y se puede activar si el presupuesto lo permite.
- **RDS no incluye TimescaleDB:** las series detalladas se quedan en el fog. En la nube se usan tablas particionadas por mes, suficientes para 1,4 millones de filas/año.
- **Instancia pequeña:** las consultas analíticas muy pesadas se ejecutan en horarios de baja carga o se exportan; si crece el uso, se pasa a una instancia mayor o a una réplica de lectura.

**Costo de esta decisión:** ≈ **14 USD/mes** (db.t4g.micro, 20 GB gp3, una zona, backups de 7 días).

## 5. Decisión C: entrada, red y seguridad

> **Decisión:** un **ALB público** con HTTPS recibe el tráfico y lo pasa a Fargate. Fargate tiene IP pública, pero su security group solo acepta tráfico del ALB. RDS está en subred privada y solo acepta conexiones de Fargate. **No se usa NAT Gateway ni API Gateway en el piloto.**

### Qué se descartó y por qué

| Opción | Costo adicional | Motivo para descartarla |
| --- | --- | --- |
| API Gateway + VPC Link | ≈ 18–20 USD/mes | No aporta nada indispensable al piloto y agrega complejidad |
| NAT Gateway | ≈ 33 USD/mes | Fargate puede salir con IP pública restringida por security group |
| RDS con IP pública | 0 USD | Expone datos personales y rompe el mínimo privilegio |
| ALB + security groups | Incluido en el costo | TLS, health checks, ruteo simple y métricas |

**Figura 2.** *Despliegue: ALB público, Fargate con IP pública restringida y RDS en subred privada.*

### El funcionamiento de la red en 3 pasos

1. **Internet → ALB (443):** el balanceador de carga recibe las conexiones seguras mediante HTTPS y utiliza un certificado de ACM para gestionar el cifrado TLS.

2. **ALB → Fargate (8080):** el ALB envía las solicitudes a los contenedores de Fargate. Su grupo de seguridad permite únicamente el tráfico proveniente del ALB, evitando el acceso directo desde Internet.

3. **Fargate → RDS (5432):** los contenedores de Fargate se conectan a la base de datos RDS. El grupo de seguridad de RDS solo permite conexiones desde Fargate y, al no contar con una IP pública, la base de datos permanece protegida del acceso externo.

### Seguridad mínima del piloto

- *Usuarios:* autenticación mediante Cognito con OAuth 2.1/OIDC y uso de MFA para las cuentas de administración.
- *Secretos:* almacenamiento de credenciales en Secrets Manager con renovación automática, evitando guardar contraseñas o claves en Git.
- *Datos:* protección de la información mediante cifrado KMS en reposo y TLS 1.2 o superior durante la transmisión; se utiliza mTLS para la comunicación entre fog y API.
- *Accesos:* asignación de permisos IAM limitados a las funciones necesarias y registro de actividades mediante CloudTrail para su revisión.
- *Base de datos:* implementación en una subred privada, con copias de seguridad conservadas durante 7 días y snapshots en momentos importantes del proyecto.

## 6. Matriz de decisión y costo

La matriz usa puntajes de 1 a 5 (5 = mejor). Los pesos son un juicio del equipo y se pueden ajustar con el docente.

| Criterio | Peso | API: EC2 | API: Fargate | API: Lambda | BD: RDS | BD: PG EC2 | BD: DynamoDB |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Costo | 25 % | 4 | 4 | 5 | 4 | 4 | 5 |
| Esfuerzo de operación | 25 % | 2 | 4 | 3 | 5 | 2 | 4 |
| Encaje con el proyecto | 20 % | 5 | 4 | 2 | 5 | 4 | 2 |
| Seguridad y LOPDP | 15 % | 3 | 4 | 4 | 4 | 3 | 4 |
| Escalabilidad | 15 % | 3 | 4 | 5 | 3 | 3 | 5 |
| **Total** | **100 %** | **3,40** | **4,00** | **3,75** | **4,30** | **3,20** | **4,00** |

**Cómo se lee:** Fargate gana la comparación de API con 4,00 y RDS gana la de base de datos con 4,30. La decisión se mantiene aunque el costo suba de peso; solo cambiaría si el costo dominara por completo y el equipo aceptara reescribir el BFF como funciones.

### Costo mensual completo

Precios públicos aproximados de AWS, región us-east-1, bajo demanda, octubre de 2026. No es una cotización.

| Servicio | Configuración | USD/mes |
| --- | --- | --- |
| ECS Fargate | 0,5 vCPU + 1 GB; promedio 1,1 tareas | 19,80 |
| Application Load Balancer | 1 ALB, tráfico bajo | 17,90 |
| RDS PostgreSQL | db.t4g.micro, 20 GB gp3, una zona, backups 7 días | 14,00 |
| IP pública de Fargate | 1 tarea | 3,70 |
| S3 | 10 GB + solicitudes | 1,70 |
| CloudWatch | logs, métricas y alarmas | 4,20 |
| Secrets Manager | 4 secretos | 1,50 |
| KMS | 1 llave + solicitudes | 1,10 |
| Route 53 | 1 zona + consultas | 0,90 |
| ECR | 2 GB de imágenes | 0,30 |
| Transferencia de datos | ≈ 20 GB/mes | 2,00 |
| Entrenamiento de modelos | ocasional, spot | 4,00 |
| **Subtotal** | | **71,10** |
| Contingencia 10 % | | 7,11 |
| **Total estimado** | | **≈ 78,21** |

**Holgura:** 83,33 − 78,21 ≈ **5,12 USD/mes** (≈ 123 USD en 24 meses). La cifra de 54 USD de la versión inicial no incluía ALB, IP pública, KMS, Route 53 ni contingencia; por eso el costo completo es ≈ 78 USD.

### Escenarios

| Escenario | Cambio | USD/mes | ¿Cabe en 83 USD? |
| --- | --- | --- | --- |
| Base | 1 tarea Fargate promedio | ≈ 78 | Sí |
| Pico | 2 tareas Fargate 24/7 | ≈ 96 | No; usar autoscaling |
| Alta disponibilidad | RDS Multi-AZ | ≈ 92 | No; posponer |
| Aislamiento de red | NAT Gateway + subred privada | ≈ 111 | No; posponer |

**Control:** AWS Budgets con alertas en 70, 80 y 83 USD/mes; etiquetas project, env, owner y cost-center en todos los recursos.

## 7. Riesgos y mitigaciones

| ID | Riesgo | Probabilidad | Impacto | Mitigación |
| --- | --- | --- | --- | --- |
| R-01 | RDS en una sola zona falla | Media | Alto | El fog sigue capturando; RTO ≤ 4 h; evaluar Multi-AZ |
| R-02 | Fargate con IP pública expuesta | Baja | Alto | Security group solo desde ALB; sin puertos abiertos; escaneo |
| R-03 | El costo supera 83 USD/mes | Media | Medio | AWS Budgets; autoscaling 1–2 tareas; plan B en EC2 o Lambda |
| R-04 | Fuga de credenciales | Baja | Alto | Secrets Manager, GitHub OIDC y escaneo de secretos |
| R-05 | Backup no probado | Media | Alto | Restauración trimestral y snapshots en hitos |
| R-06 | RDS no soporta TimescaleDB | Alta | Bajo | Particionado nativo; TimescaleDB se queda en el fog |

**Riesgo aceptado:** RDS single-AZ. Se documenta y se reevalúa si el presupuesto o la criticidad cambian.

## 8. Integración con los Equipos 1, 2 y 3

Este documento no reemplaza el trabajo de los otros equipos: fija los nombres y las responsabilidades para que el H2 integrado no tenga contradicciones.

| Equipo | Qué necesitamos de ellos | Qué les entregamos |
| --- | --- | --- |
| **Equipo 1: inventario** | Confirmar los nombres de los servicios | Lista: VPC, ALB, ECS Fargate, ECR, RDS, S3, Cognito, Secrets Manager, KMS, CloudWatch, CloudTrail, Route 53 y AWS Budgets |
| **Equipo 2: clasificación** | Usar la misma clasificación IaaS/PaaS/SaaS | Fargate = PaaS/CaaS; RDS, S3, ALB, Secrets Manager y KMS = PaaS; Cognito = SaaS/IDaaS; VPC = IaaS de red; fog = on-premise |
| **Equipo 3: responsabilidades y seguridad** | Validar la matriz RACI y los controles | AWS administra hardware, hipervisor, runtime de Fargate y motor de RDS; AgriSmart administra esquema, datos, IAM, secretos, backups y LOPDP |

**Tres reglas para la integración:**

1. Usar los mismos nombres de servicio en las tablas de los cuatro equipos.
2. No agregar API Gateway ni NAT Gateway sin actualizar el ADR-015 y la matriz de costos.
3. Mantener el flujo **fog → API → RDS**; nunca fog → RDS directo.

## 9. Validación y evidencias

| Prueba | Criterio | Evidencia |
| --- | --- | --- |
| Idempotencia | Enviar la misma lectura 10 veces y obtener 1 fila | Log de prueba |
| Carga | 1.000 lecturas por lote y 50 usuarios; p95 < 2 s | Reporte de k6/Locust |
| Desconexión | Cortar fog–nube 6 h; 0 pérdidas y 0 duplicados | Simulación de red |
| Seguridad | Solo 443 público; RDS privado; TLS 1.2+ | nmap / SSL Labs |
| Restauración | Restaurar un snapshot en ≤ 4 h | Reporte de RDS |
| Costo | Gasto real ≤ 83 USD/mes | Cost Explorer / Budgets |

## 10. Conclusión

El piloto usa **ECS Fargate** para la API y el BFF, **PostgreSQL en RDS** para los datos consolidados y un **ALB** como entrada pública. El fog sigue siendo la garantía de captura y alerta cuando no hay internet. El costo completo es ≈ **78 USD/mes** con contingencia, dentro del presupuesto de ≈ 83 USD/mes.

La decisión es defendible porque:

- reduce la operación que asumen los estudiantes;
- conserva la portabilidad de Docker y PostgreSQL;
- mantiene la base de datos fuera de internet;
- respeta los ADR del proyecto;
- tiene un plan B claro: EC2, Lambda, Multi-AZ o API Gateway si cambian las condiciones.

### Próximos pasos

1. Validar costos con la calculadora de AWS y capturar la evidencia.
2. Revisar el documento con los Equipos 1, 2 y 3.
3. Integrar la sección 4 en docs/nube-H2.md.
4. Implementar el esqueleto de Terraform y el esquema inicial de la base.
5. Abrir la PR y solicitar revisión.

| Término | Significado |
| --- | --- |
| **ADR** | Architecture Decision Record: registro corto de una decisión y sus consecuencias |
| **ALB** | Application Load Balancer: balanceador HTTP/HTTPS administrado de AWS |
| **C4** | Modelo para documentar arquitectura en cuatro niveles: contexto, contenedores, componentes y código |
| **Fargate** | Servicio de AWS que ejecuta contenedores sin administrar servidores |
| **Idempotencia** | Propiedad por la que repetir una operación no cambia el resultado |
| **LOPDP** | Ley Orgánica de Protección de Datos Personales de Ecuador |
| **mTLS** | TLS mutuo: ambos extremos se autentican con certificado |
| **Multi-AZ** | Copia de la base de datos en otra zona de disponibilidad |
| **PaaS** | Platform as a Service: el proveedor administra infraestructura y runtime |
| **RDS** | Relational Database Service: base de datos administrada de AWS |
| **Single-AZ** | Una sola zona de disponibilidad; más barato, pero sin failover automático |
| **VPC** | Virtual Private Cloud: red aislada dentro de AWS |
