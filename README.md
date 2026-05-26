# EasyPark
### Sistema de Parqueaderos Privados en Tiempo Real

**Autor:** Diego Alejandro Vega Vanegas
**Universidad:** Universidad Católica de Oriente
**Asignatura:** Ingeniería de Software 2

---

> EasyPark conecta conductores que necesitan dónde aparcar con propietarios de espacios privados que quieren generar ingresos. Reservas en tiempo real, pagos digitales y mapa interactivo, todo en una sola plataforma web.

---

## Índice

- [¿Qué es EasyPark?](#qué-es-easypark)
- [Contexto del problema](#contexto-del-problema)
- [Roles del sistema](#roles-del-sistema)
- [Requisitos funcionales](#requisitos-funcionales)
- [Requisitos no funcionales](#requisitos-no-funcionales)
- [Funcionalidades críticas](#funcionalidades-críticas)
- [Atributos de calidad](#atributos-de-calidad)
- [Restricciones técnicas](#restricciones-técnicas)
- [Restricciones de negocio](#restricciones-de-negocio)
- [Herramientas y tecnologías](#herramientas-y-tecnologías)
- [Arquitectura del sistema](#arquitectura-del-sistema)
- [Diagramas del proyecto](#diagramas-del-proyecto)
- [Cómo correr el proyecto](#cómo-correr-el-proyecto)

---

## ¿Qué es EasyPark?

**EasyPark** nació de una necesidad muy real: ¿cuántas veces has dado vueltas y vueltas buscando dónde dejar el carro? La plataforma es una aplicación web reactiva que conecta conductores con dueños de parqueaderos privados (garajes de casas u otros espacios privados) dentro de una ciudad.

La idea es simple pero poderosa: si tienes un espacio vacío, ganas dinero. Si necesitas parqueadero, lo encuentras en segundos. Todo desde el celular o el computador, sin llamadas, sin intermediarios y sin sorpresas.

El sistema permite buscar espacios disponibles en tiempo real, reservarlos con bloqueo temporal a través de Redis, procesar pagos de forma segura y recibir notificaciones automáticas en cada paso del proceso.

Técnicamente está construido sobre **Spring Boot 3.3 + Angular 17 + PostgreSQL 17 + Redis + WebSocket + Firebase FCM + Mercado Pago**, siguiendo una **arquitectura hexagonal** que mantiene la lógica de negocio completamente aislada de los detalles técnicos, lo que hace el sistema mucho más fácil de mantener y evolucionar.

---

## Contexto del problema

El problema es cotidiano y lo vive mucha gente: los dueños de parqueaderos o espacios privados tienen la oportunidad de generar ingresos alquilando celdas bien ubicadas para guardar vehículos, pero no tienen cómo llegar a los conductores que los necesitan.

Por el otro lado, muchos conductores se enfrentan a diario al estrés de no saber dónde dejar su vehículo, especialmente en las noches, durante las vacaciones o en zonas de alta demanda. Terminan estacionando en lugares indebidos, caminando distancias largas o simplemente dando vueltas sin éxito.

**EasyPark resuelve exactamente eso:** conecta a esas dos personas de forma digital, segura y sencilla. Sin apps que instalar, sin llamadas para hacer, sin incertidumbre sobre si el espacio estará disponible cuando llegues.

---

## Roles del sistema

El sistema está diseñado para tres tipos de usuarios, cada uno con responsabilidades y permisos bien definidos:

| Rol | Descripción |
|-----|-------------|
| **Conductor** | Busca parqueaderos en el mapa, hace reservas y paga desde la plataforma. Es el usuario principal del sistema. |
| **Dueño del parqueadero** | Publica su espacio, gestiona la disponibilidad y decide si acepta o rechaza las reservas que llegan. |
| **Administrador** | Es el guardián del sistema. Aprueba los parqueaderos antes de que sean visibles, gestiona usuarios y monitorea que todo funcione correctamente. |

---

## Requisitos funcionales

### CU001 — Crear Administrador

El proceso de creación de administradores es controlado y seguro. El sistema debe:

- Permitir al administrador iniciar sesión en el sistema
- Presentar un formulario con campos obligatorios: nombre, apellido, tipo y número de identificación, correo y contraseña
- Validar que el correo sea único en el sistema y que la contraseña cumpla los criterios de seguridad (mínimo 8 caracteres, al menos una mayúscula, un número y un carácter especial)
- Crear la cuenta si todas las validaciones son exitosas
- Confirmarle al administrador que su cuenta fue creada correctamente

---

### CU002 — Iniciar Sesión

El acceso al sistema debe ser claro y sin fricciones. El sistema debe:

- Mostrar un formulario de inicio de sesión sencillo
- Validar las credenciales ingresadas contra la base de datos
- Si son válidas, redirigir al usuario a la página principal según su rol (conductor, dueño o admin)
- Si son inválidas, mostrar un mensaje de error claro que le indique al usuario qué salió mal

---

### CU003 — Registro de Usuario

Cualquier persona puede crear su cuenta fácilmente. El sistema debe:

- Permitir que el usuario se registre con nombre, apellido, contraseña, tipo de vehículo, placa, celular y correo
- Validar que el correo sea único y que la contraseña tenga el formato seguro requerido

---

### CU004 — Olvidar Contraseña

Si un usuario olvida su contraseña, no debería ser el fin del mundo. El sistema debe:

- Mostrar un formulario donde el usuario ingrese su correo o nombre de usuario
- Enviar automáticamente un token de restablecimiento al correo registrado
- Permitir al usuario restablecer su contraseña usando ese token
- Validar el token antes de permitir cualquier cambio, para evitar usos malintencionados

---

### CU009 — Búsqueda *(Reto Técnico)*

La búsqueda es el corazón de la experiencia del conductor. El sistema debe:

- Buscar parqueaderos disponibles usando el GPS del dispositivo, con un radio automático de 1 km
- Permitir filtrar por ubicación, horario de atención y tipo de vehículo
- Mostrar en tiempo real las celdas disponibles dentro del parqueadero seleccionado, sin necesidad de recargar la página

---

### CU010 — Reserva *(Reto Técnico)*

Hacer una reserva debe ser rápido y sin complicaciones. El sistema debe:

- Permitir completar el formulario de reserva con placa, tipo de vehículo, hora de entrada y hora de salida estimada
- Validar la disponibilidad de la celda en el horario solicitado antes de confirmar
- Asignar la celda y generar un comprobante digital con código QR que sirva como identificación al llegar
- Enviar una notificación automática al conductor 1 hora antes del inicio de la reserva
- Enviar una segunda notificación 15 minutos antes del fin del tiempo reservado

---

### CU011 — Gestión de Reserva *(Reto Técnico)*

El dueño del parqueadero tiene el control sobre las reservas. El sistema debe:

- Permitir al dueño aceptar una reserva entrante
- Permitir al dueño rechazar una reserva si lo considera necesario
- Si el conductor no completa el pago dentro del tiempo límite, liberar el espacio automáticamente para que otro usuario pueda reservarlo
- Notificar al conductor el estado de su reserva, tanto si fue confirmada como si fue rechazada

---

### CU012 — Ofrecer Parqueadero

Publicar un parqueadero debe ser tan sencillo como llenar un formulario. El sistema debe:

- Permitir registrar el parqueadero con dirección, horarios de atención, tipos de vehículos admitidos, cantidad de celdas disponibles y tarifa por hora
- Validar que la dirección no esté duplicada en el sistema y que los horarios ingresados sean coherentes
- Enviar el parqueadero al administrador para su aprobación antes de que sea visible en el mapa
- Darle al propietario la posibilidad de editar o desactivar su espacio en cualquier momento

---

### CU013 — Pagos *(Crítico)*

El pago es el punto más sensible del flujo. El sistema debe:

- Procesar pagos de forma segura a través de la pasarela Mercado Pago, directamente desde la web, antes de que el conductor pueda hacer uso del parqueadero

---

### CU014 — Gestión de Facturación

Una vez confirmado el pago, el conductor necesita su comprobante. El sistema debe:

- Generar y enviar automáticamente una factura digital con código QR que el conductor pueda presentar al llegar al parqueadero para verificar su identidad

---

### CU015 — Notificaciones

La comunicación en tiempo real es clave para que el sistema funcione sin fricciones. El sistema debe:

- Notificar al dueño del parqueadero inmediatamente cuando alguien reserve su espacio
- Bloquear automáticamente el espacio reservado en cuanto se confirme la reserva, para evitar que dos personas reserven la misma celda al mismo tiempo

---

### CU016 — Gestión de Multas

Si alguien se pasa del tiempo acordado, el sistema lo detecta y actúa. El sistema debe:

- Detectar automáticamente cuando un vehículo excede el tiempo de reserva
- Calcular la multa de forma proporcional al tiempo excedido
- Notificar al conductor el monto de la multa y el plazo máximo para pagarla
- Bloquear al usuario para que no pueda hacer nuevas reservas hasta que la multa esté saldada
- Permitir al administrador revisar y ajustar multas en casos excepcionales donde lo amerite

---

### CU017 — Pago de Reserva / Multa

El proceso de pago debe ser flexible y confiable. El sistema debe:

- Aceptar pagos con tarjeta de crédito o débito, billetera virtual o transferencia bancaria
- Generar automáticamente un recibo digital una vez confirmado el pago
- Si el pago falla por algún motivo, notificar al usuario de inmediato y darle la opción de reintentar sin perder la reserva

---

## Requisitos no funcionales

### Rendimiento y Escalabilidad

El sistema debe responder en menos de **3 segundos** tanto para búsquedas de parqueaderos como para la confirmación de pagos. Además, debe estar diseñado para escalar y admitir nuevos usuarios y nuevas ciudades sin necesidad de reescribir el código desde cero.

### Usabilidad

La interfaz debe ser tan intuitiva que cualquier persona pueda usarla sin necesitar capacitación. Incluye soporte para múltiples idiomas y está optimizada tanto para móviles como para computadores de escritorio, con un diseño completamente responsive.

### Seguridad

La autenticación se maneja con JWT stateless. Cada rol tiene acceso únicamente a las funcionalidades que le corresponden. Las contraseñas se cifran con BCrypt y nunca se almacenan en texto plano. Todo el manejo de datos personales cumple con la **Ley 1581 de 2012 (Habeas Data)** de Colombia.

### Disponibilidad y Fiabilidad

El sistema apunta a una disponibilidad mínima del **99% del tiempo**. Las copias de seguridad de la base de datos son automáticas y todas las operaciones críticas quedan registradas en el log para facilitar la auditoría y el diagnóstico de problemas.

---

## Funcionalidades críticas

Estas son las funcionalidades sin las cuales EasyPark simplemente no puede existir. Si alguna de estas falla, el sistema pierde su propósito:

| ID | Funcionalidad | Prueba de concepto |
|----|---------------|-------------------|
| CU-009-RF1 | Buscar parqueaderos por GPS en radio de 1 km | Sí |
| CU-010-RF1 | Registrar placa, tipo de vehículo y periodo de entrada/salida | No (pendiente) |
| CU-010-RF3 | Asignar celda y generar comprobante con código QR | Sí |
| CU-011-RF3 | Liberar espacio automáticamente si no se paga a tiempo | Sí — requiere jobs/watchers |
| CU-011-RF4 | Notificar al usuario el estado de su reserva | Sí |
| CU-013-RF1 | Pago seguro mediante Mercado Pago | Sí — requiere integración API |
| CU-014-RF1 | Factura con código QR para verificar identidad | Sí |

---

## Atributos de calidad

Para priorizar los atributos de calidad del sistema se utilizó un **mapa de empatía** con los tres actores principales. Cada uno votó según sus necesidades reales y el resultado fue ponderado para reflejar lo que de verdad importa en la práctica.

### Priorización final

| Posición | Atributo de Calidad | Peso Ponderado |
|----------|---------------------|----------------|
| 1 | Seguridad | 10.6% |
| 2 | Rendimiento | 10.3% |
| 3 | Usabilidad | 9.9% |
| 4 | Disponibilidad | 9.9% |
| 5 | Costo | 8.4% |
| 6 | Escalabilidad | 7.7% |
| 7 | Confiabilidad | 7.7% |
| 8 | Capacidad para ser soportado | 7.7% |
| 9 | Capacidad para ser administrado | 7.0% |
| 10 | Capacidad para ser mantenido | 5.9% |
| 11 | Interoperabilidad | 5.5% |
| 12 | Internacionalización | 5.5% |
| 13 | Capacidad para ser desplegado | 4.0% |

### Top 10 escenarios de calidad priorizados

Estos son los escenarios concretos que más peso tienen a la hora de evaluar si el sistema cumple con los estándares de calidad esperados:

| Código | Atributo | Escenario |
|--------|----------|-----------|
| ESC-CAL-SEG-001 | Seguridad | Ingreso al sistema de forma exitosa |
| ESC-CAL-SEG-002 | Seguridad | Intento fallido por contraseña incorrecta |
| ESC-CAL-SEG-0022 | Seguridad | Propietario accede solo a sus propias funciones |
| ESC-CAL-SEG-0023 | Seguridad | Administrador accede a la gestión global del sistema |
| ESC-CAL-SEG-0031 | Seguridad | Inicio de sesión con Gmail / Facebook / Office 365 |
| ESC-CAL-SEG-0041 | Seguridad | Datos de tarjeta cifrados durante el pago |
| ESC-CA-USA-0001 | Usabilidad | Conductor completa reserva en menos de 3 pasos |
| ESC-CA-USA-0022 | Usabilidad | Mensaje claro cuando los datos son inválidos |
| ESC-CA-USA-0023 | Usabilidad | Confirmación visible al publicar un parqueadero |
| ESC-CA-USA-0033 | Usabilidad | Formulario adaptable en móvil sin desplazamientos excesivos |

---

## Restricciones técnicas

Estas restricciones no son caprichos: cada una tiene una razón de ser bien justificada.

| Tipo | Restricción | Por qué importa |
|------|-------------|-----------------|
| Tecnología base | Accesible desde navegador sin instalación | Un conductor urgente no puede ponerse a instalar apps en medio de la calle |
| Tecnología base | Adaptable a móviles y computadores | La mayoría de los conductores usarán el celular mientras buscan parqueadero |
| Compatibilidad | Compatible con Chrome, Edge y Firefox | Son los navegadores más usados en Colombia según estadísticas de uso |
| Concurrencia | Soportar múltiples usuarios simultáneos | En hora pico hay decenas de personas conectadas buscando espacio al mismo tiempo |
| Tiempo real | Disponibilidad actualizada al instante | Evitar que dos personas intenten reservar la misma celda simultáneamente |
| Disponibilidad | Sistema disponible casi 24/7 | Alguien puede necesitar parqueadero a las 11pm un domingo; el sistema debe estar ahí |
| Arquitectura | Arquitectura en capas o hexagonal | Si el código es un desorden, cada cambio pequeño se convierte en un problema grande |
| Desarrollo | Aplicar principios SOLID | Código fácil de cambiar, probar y entender por cualquier miembro del equipo |
| Diseño | Bajo acoplamiento entre módulos | Un cambio en el módulo de pagos no debería romper el módulo de reservas |
| Metodología | Metodologías ágiles | Permite adaptarse a los cambios del proyecto y entregar valor de forma progresiva |

---

## Restricciones de negocio

El proyecto también tiene limitaciones del mundo real que hay que reconocer y gestionar:

| Tipo | Restricción | Plan de acción |
|------|-------------|----------------|
| Humano | Disponibilidad limitada por responsabilidades académicas | Dividir tareas por módulos, manejar cronogramas semanales y priorizar lo crítico primero |
| Tiempo | Calendario académico con fechas fijas | Scrum, MVP primero, pruebas continuas desde el inicio |
| Legal | Normativas de alquiler de espacios en Colombia | Investigar las normativas locales e incluir términos y condiciones claros en la plataforma |
| Legal | Ley 1581 de 2012 — Habeas Data | Políticas de privacidad visibles, cifrado de contraseñas, control de acceso estricto por roles |
| Legal | Seguridad en pagos digitales (PCI-DSS) | Nunca almacenar datos de tarjetas en el servidor; siempre generar comprobantes digitales |
| Tecnológico | Dependencia de GPS e internet | GPS es opcional; también se puede buscar manualmente ingresando la dirección |
| Seguridad | Riesgo de accesos no autorizados | Validación tanto en frontend como en backend, control de roles estricto en todos los endpoints |
| Presupuesto | Desembolsos por hitos aprobados | Hitos pequeños y frecuentes con comunicación constante con el cliente para evitar sorpresas |

---

## Herramientas y tecnologías

### Frontend

Para el frontend se evaluaron las dos opciones más populares del ecosistema JavaScript:

| Herramienta | Estado | Por qué se eligió o descartó |
|-------------|--------|------------------------------|
| **Angular 17** | Seleccionado | Estructura organizada, tipado fuerte con TypeScript e ideal para manejar el mapa interactivo y las reservas en tiempo real con WebSocket |
| React.js | No seleccionado | Se priorizó la robustez estructural de Angular sobre la flexibilidad más libre de React para este tipo de proyecto |

---

### Backend

| Herramienta | Estado | Por qué se eligió o descartó |
|-------------|--------|------------------------------|
| **Spring Boot 3.3** | Seleccionado | El equipo lo conoce bien, tiene todo lo que EasyPark necesita (seguridad, WebSocket, JPA) y se adapta perfectamente a la arquitectura hexagonal |
| Node.js | No seleccionado | El equipo tiene mayor experiencia en Java; cambiar de lenguaje habría agregado riesgo innecesario |
| Django | No seleccionado | El equipo no tiene experiencia suficiente en Python como para usarlo en un proyecto de esta envergadura |

---

### Hosting

| Herramienta | Estado | Por qué se eligió o descartó |
|-------------|--------|------------------------------|
| **Railway** | Seleccionado | Fácil de configurar, costos accesibles para un proyecto académico y tiene soporte integrado tanto para PostgreSQL como para Redis |
| AWS | No seleccionado | El pago por uso puede ser impredecible y la curva de aprendizaje es muy alta para el alcance actual del proyecto |
| Google Cloud / Azure | No seleccionado | Más complejo y costoso de gestionar en esta etapa del proyecto |

---

### Base de datos

| Herramienta | Estado | Por qué se eligió o descartó |
|-------------|--------|------------------------------|
| **PostgreSQL 17** | Seleccionado | Open source, robusto, el equipo ya tiene experiencia con él y garantiza integridad de datos con transacciones ACID |
| OracleSQL | No seleccionado | Costo muy alto si el sistema empieza a crecer; no viable para un proyecto académico |
| NoSQL | No seleccionado | Puede generar duplicidad de información; la flexibilidad que ofrece no es necesaria para la estructura de datos de EasyPark |

---

### Otras tecnologías clave

| Tecnología | Estado | Qué hace en EasyPark |
|------------|--------|----------------------|
| **Redis** | Seleccionado | Caché de disponibilidad y bloqueo distribuido de celdas — implementa un mutex de 5 minutos para evitar la doble reserva |
| **Spring Security + JWT** | Seleccionado | Autenticación stateless y control de acceso por rol en todos los endpoints |
| **GitHub Actions** | Seleccionado | CI/CD: ejecuta las pruebas automáticamente en cada cambio de código |
| **WebSocket (STOMP)** | Seleccionado | Actualización del mapa en tiempo real sin que el usuario tenga que recargar la página |
| **Firebase Cloud Messaging** | Seleccionado | Notificaciones push gratuitas tanto para conductores como para propietarios |
| **Mercado Pago** | Seleccionado | Pasarela de pagos líder en Colombia y Latinoamérica, con alta confianza del usuario local |
| **Google Maps Platform** | Seleccionado | Mapa interactivo con geolocalización precisa para encontrar parqueaderos cercanos |
| REST API | No seleccionado | No permite actualizaciones en tiempo real; el usuario tendría que recargar la página constantemente |
| Stripe | No seleccionado | Poco conocido en Colombia, lo que genera desconfianza a la hora de pagar |

---

### Estilos arquitectónicos aplicados

EasyPark no se construye sobre una sola arquitectura sino sobre una combinación que se complementa bien:

| Arquitectura | Estado | Por qué se aplica en EasyPark |
|--------------|--------|-------------------------------|
| **Hexagonal (Ports & Adapters)** | Aplicada | La lógica de negocio vive en el centro, completamente aislada de la base de datos y los servicios externos. Si mañana se cambia Mercado Pago por otra pasarela, solo se modifica el adaptador correspondiente |
| **En Capas** | Aplicada | Presentación → lógica de negocio → acceso a datos. Cada capa tiene una responsabilidad clara y bien delimitada |
| **Microservicios** | Aplicada | Módulos independientes para usuarios, reservas, pagos y disponibilidad; cada uno puede evolucionar por separado |
| **Event-Driven** | Aplicada | La reserva dispara una notificación, que dispara la actualización del mapa, todo como una cadena de eventos desacoplados |
| **API-First** | Aplicada | Los endpoints se diseñaron pensando en Angular desde el primer día, no como una adaptación posterior |
| **Contenedores (Docker)** | Aplicada | Garantiza consistencia entre el entorno de desarrollo y el de producción, eliminando el clásico "en mi máquina sí funciona" |

---

## Arquitectura del sistema

### Diagrama de capas

```
┌──────────────────────────────────────────────────────┐
│             CAPA DE PRESENTACIÓN                     │
│  Angular SPA · Google Maps · WebSocket Client        │
│  Dashboard Conductor / Dueño / Admin                 │
└─────────────────────┬────────────────────────────────┘
                      │ HTTP/HTTPS · WebSocket
┌─────────────────────▼────────────────────────────────┐
│          CAPA DE API GATEWAY / SEGURIDAD             │
│  JwtAuthFilter · JwtService · SecurityConfig         │
│  BCrypt · CORS · Control de roles por endpoint       │
└─────────────────────┬────────────────────────────────┘
                      │
┌─────────────────────▼────────────────────────────────┐
│          CAPA DE LÓGICA DE NEGOCIO                   │
│  AuthController · ParqueaderoController              │
│  ReservaService · RedisParqueaderoService            │
│  WebSocketHandler                                    │
└─────────────────────┬────────────────────────────────┘
                      │
┌─────────────────────▼────────────────────────────────┐
│            CAPA DE ACCESO A DATOS                    │
│  UsuarioRepository · ParqueaderoRepository           │
│  ReservaRepository · RedisTemplate · JPA/Hibernate   │
└─────────────────────┬────────────────────────────────┘
                      │
┌─────────────────────▼────────────────────────────────┐
│        CAPA DE INFRAESTRUCTURA / PERSISTENCIA        │
│  PostgreSQL 17 · Redis · Firebase FCM · Mercado Pago │
└──────────────────────────────────────────────────────┘
```

---

### Flujo completo de una reserva

Este es el recorrido completo desde que un conductor inicia sesión hasta que llega al parqueadero con su QR en mano:

```
1.  El conductor inicia sesión y recibe su token JWT
2.  Carga el mapa con los parqueaderos disponibles cerca a su ubicación
3.  Se conecta al WebSocket para recibir actualizaciones en tiempo real
4.  Selecciona el parqueadero que le interesa e inicia el proceso de reserva
5.  Redis bloquea la celda por 5 minutos con un mutex atómico para evitar conflictos
6.  El WebSocket notifica a todos los usuarios conectados y el mapa se actualiza automáticamente
7.  El dueño acepta la reserva y el conductor recibe la notificación de confirmación
8.  El conductor realiza el pago a través de Mercado Pago
9.  Mercado Pago envía el webhook al backend y este confirma la reserva definitivamente
10. PostgreSQL actualiza el estado de la reserva y Redis libera el bloqueo de la celda
11. Firebase FCM notifica tanto al conductor como al dueño del parqueadero
12. El conductor llega al parqueadero y presenta el código QR para verificar su identidad
```

---

### Estructura de paquetes — Backend

```
co.edu.uco.parking/
├── controller/        → Recibe y responde peticiones HTTP y WebSocket
├── security/          → Spring Security, filtros JWT y configuración de BCrypt
├── config/            → Configuración de Redis, CORS, WebSocket y beans de Spring
├── dto/               → Objetos de transferencia de datos (request y response)
├── entity/            → Entidades JPA: Usuario, Parqueadero, Reserva, etc.
├── repository/        → Interfaces Spring Data JPA para acceso a la base de datos
├── service/           → Interfaces de los servicios (puertos de entrada en arquitectura hexagonal)
├── service/impl/      → Implementaciones concretas de la lógica de negocio
└── websocket/         → Manejadores de mensajes en tiempo real vía STOMP
```

---

### Estructura de módulos — Frontend Angular 17

```
easy-park-frontend/
├── src/app/
│   ├── core/
│   │   ├── services/      → Servicios HTTP, autenticación y WebSocket
│   │   └── guards/        → Guards de rutas con control por rol
│   ├── shared/
│   │   ├── models/        → Interfaces TypeScript compartidas
│   │   └── components/    → Componentes reutilizables en toda la app
│   └── features/
│       ├── auth/          → Login, registro y recuperación de contraseña
│       ├── conductor/     → Mapa interactivo, búsqueda, reservas y pagos
│       ├── dueno/         → Publicar parqueadero y gestionar reservas entrantes
│       └── admin/         → Panel de administración global del sistema
```

---

## Cómo correr el proyecto

### Requisitos previos

Antes de levantar el proyecto asegurate de tener instalado y funcionando lo siguiente:

- Java 21 (OpenJDK)
- PostgreSQL 17 corriendo en el puerto `5432`
- Redis corriendo en el puerto `6379`
- Eclipse IDE con Spring Tools 4 (para el backend)
- Node.js 20+ y Angular CLI 17+ (para el frontend)

---

### Levantar el backend

```bash
# Iniciar los servicios en Windows
net start postgresql-x64-17
net start Redis

# Opción 1 — Desde Eclipse:
# Boot Dashboard → seleccionar easy-park → Start

# Opción 2 — Con Maven desde la terminal:
mvn spring-boot:run
```

El backend queda disponible en `http://localhost:8080`

---

### Levantar el frontend

```bash
cd easy-park-frontend
npm install
ng serve
```

El frontend queda disponible en `http://localhost:4200`

---

### Configuración — `application.yml`

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://127.0.0.1:5432/easy_park
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update
  data:
    redis:
      host: localhost
      port: 6379

app:
  jwt:
    secret: easyPark2024SecretKeyMuyLargaParaQueSeaSegura
    expiration: 86400000   # 24 horas en milisegundos
  parking:
    lock-timeout: 300      # Tiempo de bloqueo Redis en segundos (5 minutos)
```

---

*Proyecto académico desarrollado para la asignatura Ingeniería de Software 2 — Universidad Católica de Oriente*
