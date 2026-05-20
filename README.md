# EasyPark
### Sistema de Parqueaderos Privados en Tiempo Real

**Autor:** Diego Alejandro Vega Vanegas  
**Universidad:** Universidad Católica de Oriente  
**Asignatura:** Ingeniería de Software 2 — ingenieria software  


> EasyPark conecta conductores que necesitan dónde aparcar con propietarios de espacios privados que quieren generar ingresos. Reservas en tiempo real, pagos digitales y mapa interactivo, todo en una sola plataforma web.

---

##  Índice

- [¿Qué es EasyPark?](#-qué-es-easypark)
- [Contexto del problema](#-contexto-del-problema)
- [Roles del sistema](#-roles-del-sistema)
- [Requisitos funcionales](#-requisitos-funcionales)
- [Requisitos no funcionales](#-requisitos-no-funcionales)
- [Funcionalidades críticas](#-funcionalidades-críticas)
- [Atributos de calidad](#-atributos-de-calidad)
- [Restricciones técnicas](#-restricciones-técnicas)
- [Restricciones de negocio](#-restricciones-de-negocio)
- [Herramientas y tecnologías](#-herramientas-y-tecnologías)
- [Arquitectura del sistema](#-arquitectura-del-sistema)
- [Diagramas del proyecto](#-diagramas-del-proyecto)
- [Cómo correr el proyecto](#-cómo-correr-el-proyecto)

---

##  ¿Qué es EasyPark?

**EasyPark** es una plataforma web reactiva que conecta conductores con dueños de parqueaderos privados (garajes de casas u otros espacios privados) en una ciudad. Permite buscar espacios disponibles en tiempo real, reservarlos con bloqueo temporal vía Redis, procesar pagos de forma segura y recibir notificaciones automáticas.

El sistema está construido con **Spring Boot 3.3 + Angular 17 + PostgreSQL 17 + Redis + WebSocket + Firebase FCM + Mercado Pago**, siguiendo una **arquitectura hexagonal** que separa la lógica de negocio de los detalles técnicos.

---

##  Contexto del problema

Los dueños de parqueaderos o espacios privados tienen la oportunidad de ganar dinero alquilando celdas cómodas y bien ubicadas para guardar vehículos (carros y motos). Al mismo tiempo, muchos conductores se enfrentan a diario al problema de no saber dónde dejar su vehículo, especialmente en las noches o durante las vacaciones.

EasyPark resuelve exactamente eso: conecta a esas dos personas de forma digital, segura y sencilla.

---

##  Roles del sistema

| Rol | Qué puede hacer |
|---|---|
|  **Conductor** | Busca parqueaderos en el mapa, hace reservas y paga desde la plataforma |
|  **Dueño del parqueadero** | Publica su espacio, gestiona disponibilidad y acepta o rechaza reservas |
| **Administrador** | Aprueba parqueaderos antes de que sean visibles, gestiona usuarios y monitorea el sistema |

---

## ⚙️ Requisitos funcionales

### CU001 — Crear Administrador
- **RF1** El administrador debe poder iniciar sesión en el sistema
- **RF2** Formulario con campos obligatorios: nombre, apellido, tipo y número de identificación, correo, contraseña
- **RF3** Validar correo único y contraseña segura (mínimo 8 caracteres, mayúscula, número y carácter especial)
- **RF4** Si la validación es exitosa, crear la cuenta
- **RF5** Confirmar la creación exitosa al administrador

### CU002 — Iniciar Sesión
- **RF1** Proporcionar formulario de inicio de sesión
- **RF2** Validar las credenciales ingresadas
- **RF3** Si son válidas, redirigir a la página principal según el rol
- **RF4** Si son inválidas, mostrar mensaje de error claro

### CU003 — Registro de Usuario
- **RF1** El usuario puede crear cuenta con nombre, apellido, contraseña, tipo de vehículo, placa, celular y correo
- **RF2** Validar correo único y formato seguro de contraseña

### CU004 — Olvidar Contraseña
- **RF1** Formulario para ingresar correo o nombre de usuario
- **RF2** Enviar token de restablecimiento al correo registrado
- **RF3** Permitir restablecer la contraseña usando el token
- **RF4** Validar el token antes de permitir el cambio

### CU009 — Búsqueda  *Reto Técnico*
- **RF1** Buscar parqueaderos disponibles por GPS (radio de 1 km automático)
- **RF2** Filtrar por ubicación, horario y tipo de vehículo
- **RF3** Mostrar en tiempo real las celdas disponibles del parqueadero seleccionado

### CU010 — Reserva  *Reto Técnico*
- **RF1** Completar formulario: placa, tipo de vehículo, hora de entrada y salida estimada
- **RF2** Validar disponibilidad de la celda en el horario solicitado
- **RF3** Asignar celda y generar comprobante digital con código QR
- **RF4** Enviar notificación automática 1 hora antes del inicio de la reserva
- **RF5** Enviar segunda notificación 15 minutos antes del fin

### CU011 — Gestión de Reserva  *Reto Técnico*
- **RF1** El dueño puede aceptar la reserva
- **RF2** El dueño puede rechazar la reserva
- **RF3** Si el usuario no paga en el tiempo límite, el espacio se libera automáticamente
- **RF4** El sistema notifica al usuario el estado de su reserva (confirmada o rechazada)

### CU012 — Ofrecer Parqueadero
- **RF1** Registrar parqueadero: dirección, horarios, tipos de vehículos admitidos, cantidad de celdas y tarifa por hora
- **RF2** Validar que la dirección no esté duplicada y que los horarios sean coherentes
- **RF3** El administrador debe aprobar el parqueadero antes de que sea visible en el mapa
- **RF4** El propietario puede editar o desactivar su espacio en cualquier momento

### CU013 — Pagos  *Crítico*
- **RF1** Pago seguro mediante pasarela de pagos (Mercado Pago), desde web, antes de usar el parqueadero

### CU014 — Gestión de Facturación
- **RF1** El usuario recibe factura con código QR para verificar su identidad al llegar al parqueadero

### CU015 — Notificaciones
- **RF1** El dueño recibe notificación cuando alguien reserva su espacio
- **RF2** El sistema bloquea automáticamente el espacio reservado para evitar doble reserva

### CU016 — Gestión de Multas
- **RF1** Detectar automáticamente si un vehículo excede el tiempo de reserva
- **RF2** Calcular multa proporcional al tiempo excedido
- **RF3** Notificar al usuario con el monto y el plazo máximo de pago
- **RF4** Bloquear nuevas reservas del usuario hasta que la multa sea pagada
- **RF5** El administrador puede revisar y modificar multas en casos excepcionales

### CU017 — Pago de Reserva/Multa
- **RF1** Pagar mediante tarjeta de crédito/débito, billetera virtual o transferencia bancaria
- **RF2** Generar recibo digital automáticamente tras confirmar el pago
- **RF3** En caso de pago fallido, notificar al usuario y permitir reintentar

---

##  Requisitos no funcionales

### Rendimiento y Escalabilidad
- Responder en menos de **3 segundos** para búsqueda de parqueaderos
- Los pagos deben confirmarse en menos de **3 segundos**
- Escalable para admitir nuevos usuarios y lugares sin reescribir el sistema

### Usabilidad
- Interfaz intuitiva que no requiere capacitación
- Soporte de traducción en diferentes idiomas
- Optimizada para móviles y computadores (diseño responsive)

### Seguridad
- Autenticación con usuario y contraseña mediante JWT stateless
- Funcionalidades diferenciadas por rol (CONDUCTOR, DUENO, ADMIN)
- Protección de datos personales conforme a la Ley 1581 de 2012 (Habeas Data)
- Contraseñas cifradas con BCrypt, nunca en texto plano

### Disponibilidad y Fiabilidad
- Disponibilidad mínima del **99%** del tiempo
- Copias de seguridad automáticas de la base de datos
- Todas las operaciones críticas quedan registradas en el log

---

##  Funcionalidades críticas

Estas son las funcionalidades sin las cuales EasyPark no puede existir como sistema:

| ID | Funcionalidad | PoC |
|---|---|---|
| **CU-009-RF1** | Buscar parqueaderos por GPS en radio de 1 km | ✅ Sí |
| **CU-010-RF1** | Registrar placa, tipo de vehículo y periodo de entrada/salida | ❌ No |
| **CU-010-RF3** | Asignar celda y generar comprobante con código QR | ✅ Sí |
| **CU-011-RF3** | Liberar espacio automáticamente si no se paga a tiempo | ✅ Sí — requiere jobs/watchers |
| **CU-011-RF4** | Notificar al usuario el estado de su reserva | ✅ Sí |
| **CU-013-RF1** | Pago seguro mediante Mercado Pago | ✅ Sí — requiere integración API |
| **CU-014-RF1** | Factura con código QR para verificar identidad | ✅ Sí |

---

##  Atributos de calidad

Los atributos de calidad se priorizaron mediante un **mapa de empatía** con los tres actores del sistema. Cada uno votó según sus necesidades reales y el resultado fue ponderado.

### Priorización final

| # | Atributo de Calidad | % Ponderado |
|---|---|---|
| 1 | **Seguridad** | 10.6% |
| 2 |  **Rendimiento** | 10.3% |
| 3 |  **Usabilidad** | 9.9% |
| 4 |  **Disponibilidad** | 9.9% |
| 5 |  **Costo** | 8.4% |
| 6 |  **Escalabilidad** | 7.7% |
| 7 |  **Confiabilidad** | 7.7% |
| 8 |  **Capacidad para ser soportado** | 7.7% |
| 9 |  **Capacidad para ser administrado** | 7.0% |
| 10 |  **Capacidad para ser mantenido** | 5.9% |
| 11 |  **Interoperabilidad** | 5.5% |
| 12 |  **Internacionalización** | 5.5% |
| 13 |  **Capacidad para ser desplegado** | 4.0% |

### Top 10 escenarios de calidad priorizados

| Código | Atributo | Escenario |
|---|---|---|
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

##  Restricciones técnicas

| Tipo | Restricción | Por qué importa |
|---|---|---|
| Tecnología base | Accesible desde navegador sin instalación | Un conductor urgente no puede ponerse a instalar apps |
| Tecnología base | Adaptable a móviles y computadores | La mayoría usará el celular en la calle |
| Compatibilidad | Compatible con Chrome, Edge y Firefox | Son los navegadores más usados en Colombia |
| Concurrencia | Soportar múltiples usuarios simultáneos | En hora pico hay mucha gente conectada al mismo tiempo |
| Tiempo real | Disponibilidad actualizada al instante | Evitar que dos personas reserven la misma celda |
| Disponibilidad | Sistema disponible casi 24/7 | Alguien puede necesitar parqueadero a las 11pm un domingo |
| Arquitectura | Arquitectura en capas o hexagonal | Si el código es un desorden, cada cambio se convierte en un problema |
| Desarrollo | Aplicar principios SOLID | Código fácil de cambiar, probar y entender por todo el equipo |
| Diseño | Bajo acoplamiento entre módulos | Un cambio en pagos no debería afectar el módulo de reservas |
| Metodología | Metodologías ágiles | Permite adaptarse a cambios y entregar valor progresivamente |

---

##  Restricciones de negocio

| Tipo | Restricción | Plan de acción |
|---|---|---|
| Humano | Disponibilidad limitada por responsabilidades académicas | Dividir tareas por módulos, cronogramas semanales, priorizar lo crítico |
| Tiempo | Calendario académico con fechas fijas | Scrum, MVP primero, pruebas continuas |
| Legal | Normativas de alquiler de espacios (Colombia) | Investigar normativas locales, incluir términos y condiciones claros |
| Legal | Ley 1581 de 2012 — Habeas Data | Políticas de privacidad visibles, cifrado de contraseñas, control de acceso por roles |
| Legal | Seguridad en pagos digitales (PCI-DSS) | Nunca almacenar datos de tarjetas, generar comprobantes digitales |
| Tecnológico | Dependencia de GPS e internet | GPS opcional, también se puede buscar manualmente por dirección |
| Seguridad | Riesgo de accesos no autorizados | Validación en frontend y backend, control de roles estricto |
| Presupuesto | Desembolsos por hitos aprobados | Hitos pequeños y frecuentes, comunicación constante con el cliente |

---

##  Herramientas y tecnologías

### Frontend

| Herramienta | Estado | Por qué se eligió |
|---|---|---|
| **Angular 17** | ✅ Seleccionado | Estructura organizada, tipado fuerte con TypeScript, ideal para manejar el mapa y las reservas en tiempo real |
| React.js | ❌ No seleccionado | Se priorizó la robustez estructural de Angular sobre la flexibilidad de React |

### Backend

| Herramienta | Estado | Por qué se eligió |
|---|---|---|
| **Spring Boot 3.3** | ✅ Seleccionado | El equipo lo conoce bien, tiene todo lo que EasyPark necesita (seguridad, WebSocket, JPA) y se adapta perfectamente a la arquitectura hexagonal |
| Node.js | ❌ No seleccionado | El equipo tiene mayor experiencia en Java |
| Django | ❌ No seleccionado | El equipo no tiene experiencia en Python |

### Hosting

| Herramienta | Estado | Por qué se eligió |
|---|---|---|
| **Railway** | ✅ Seleccionado | Fácil de configurar, costos accesibles para un proyecto académico, soporte integrado para PostgreSQL y Redis |
| AWS | ❌ No seleccionado | Pago por uso impredecible, curva de aprendizaje muy alta |
| Google Cloud / Azure | ❌ No seleccionado | Más complejo y costoso para la etapa actual del proyecto |

### Base de Datos

| Herramienta | Estado | Por qué se eligió |
|---|---|---|
| **PostgreSQL 17** | ✅ Seleccionado | Open source, robusto, el equipo ya tiene experiencia y garantiza integridad con transacciones ACID |
| OracleSQL | ❌ No seleccionado | Costo muy alto si el sistema crece |
| NoSQL | ❌ No seleccionado | Puede ocasionar duplicidad de información; la flexibilidad que ofrece no es necesaria para EasyPark |

### Otras tecnologías clave

| Tecnología | Estado | Qué hace en EasyPark |
|---|---|---|
| **Redis** | ✅ Seleccionado | Caché de disponibilidad y bloqueo distribuido de celdas (mutex de 5 minutos para evitar doble reserva) |
| **Spring Security + JWT** | ✅ Seleccionado | Autenticación stateless y control de acceso por rol |
| **GitHub Actions** | ✅ Seleccionado | CI/CD: pruebas automáticas en cada cambio de código |
| **WebSocket (STOMP)** | ✅ Seleccionado | Actualización del mapa en tiempo real sin que el usuario recargue |
| **Firebase Cloud Messaging** | ✅ Seleccionado | Notificaciones push gratuitas a conductores y propietarios |
| **Mercado Pago** | ✅ Seleccionado | Pasarela de pagos líder en Colombia y Latinoamérica |
| **Google Maps Platform** | ✅ Seleccionado | Mapa interactivo con geolocalización precisa |
| REST API | ❌ No seleccionado | No permite actualizaciones en tiempo real; el usuario tendría que recargar constantemente |
| Stripe | ❌ No seleccionado | Poco conocido en Colombia, genera desconfianza a la hora de pagar |

### Estilos arquitectónicos aplicados

| Arquitectura | Estado | Por qué se aplica en EasyPark |
|---|---|---|
| **Hexagonal (Ports & Adapters)** | ✅ Aplicada | La lógica de negocio vive en el centro, aislada de la base de datos y los servicios externos. Si mañana cambia Mercado Pago, solo se cambia el adaptador |
| **En Capas** | ✅ Aplicada | Presentación → lógica de negocio → acceso a datos. Cada capa tiene su responsabilidad clara |
| **Microservicios** | ✅ Aplicada | Módulos independientes: usuarios, reservas, pagos, disponibilidad |
| **Event-Driven** | ✅ Aplicada | Reserva → notificación → actualización del mapa como cadena de eventos desacoplados |
| **API-First** | ✅ Aplicada | Los endpoints se diseñaron pensando en Angular desde el inicio |
| **Contenedores (Docker)** | ✅ Aplicada | Consistencia entre entornos de desarrollo y producción |

---

## <img width="1260" height="581" alt="diagrama de capas back end drawio" src="https://github.com/user-attachments/assets/bf912e99-5e6b-4e4e-98d5-d47f3d47ce2e" />
 Arquitectura del sistema
[DAS_EasyPark.docx](https://github.com/user-attachments/files/28040785/DAS_EasyPark.docx)

### Diagrama de capas
<img width="2386" height="1067" alt="arquitectura de referencia llena drawio" src="https://github.com/user-attachments/assets/62c25a56-85e6-496b-b02d-007399502382" /><img width="791" height="811" alt="diagrama paquetes backend (1) drawio" src="https://github.com/user-attachments/assets/881f4585-c784-4048-b421-e624c9e21f61" />
<img width="891" height="441" alt="Diagrama sin título-BACK END drawio" src="https://github.com/user-attachments/assets/04faa81f-6fe8-4270-a21c-7acc5efaaf2f" />
[DAS_EasyPark.docx](https://github.com/user-attachments/files/28040777/DAS_EasyPark.docx)
<img width="1181" height="661" alt="arqueotipo de referencia drawio" src="https://github.com/user-attachments/assets/0d535920-4299-43bf-ad3b-0cb86dd70236" />
<img width="751" height="881" alt="front end paquetes drawio" src="https://github.com/user-attachments/assets/9f8d603f-7887-430b-9b31-84124c324418" />
[markdown_frontend_easypark.md](https://github.com/user-attachments/files/28040775/markdown_frontend_easypark.md)
<img width="1127" height="951" alt="DIAGRAMA DE DESPLIEGUE drawio" src="https://github.com/user-attachments/assets/991eff31-bf01-45cb-978e-b9379c814cc3" />

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

### Flujo completo de una reserva

```
1.  Conductor inicia sesión → recibe token JWT
2.  Carga el mapa con parqueaderos disponibles
3.  Se conecta al WebSocket → suscripción en tiempo real
4.  Selecciona el parqueadero e inicia la reserva
5.  Redis bloquea la celda por 5 minutos (mutex atómico)
6.  WebSocket notifica a todos los usuarios → mapa se actualiza
7.  El dueño acepta la reserva → notificación al conductor
8.  Conductor paga con Mercado Pago
9.  Mercado Pago envía webhook → backend confirma la reserva
10. PostgreSQL actualiza el estado · Redis libera el bloqueo
11. Firebase FCM notifica al conductor y al dueño
12. El conductor llega al parqueadero y muestra el QR
```

### Estructura de paquetes — Backend

```
co.edu.uco.parking/
├── controller/        → Recibe peticiones HTTP y WebSocket
├── security/          → Spring Security, filtros JWT, BCrypt
├── config/            → Redis, CORS, WebSocket, beans de Spring
├── dto/               → Objetos de transferencia (request/response)
├── entity/            → Entidades JPA (Usuario, Parqueadero, Reserva)
├── repository/        → Interfaces Spring Data JPA
├── service/           → Interfaces de los servicios (puertos de entrada)
├── service/impl/      → Implementaciones de la lógica de negocio
└── websocket/         → Manejadores de mensajes en tiempo real
```

### Estructura de módulos — Frontend Angular 17

```
easy-park-frontend/
├── src/app/
│   ├── core/
│   │   ├── services/      → Servicios HTTP, auth, WebSocket
│   │   └── guards/        → Guards de rutas por rol
│   ├── shared/
│   │   ├── models/        → Interfaces TypeScript
│   │   └── components/    → Componentes reutilizables
│   └── features/
│       ├── auth/          → Login, registro, recuperar contraseña
│       ├── conductor/     → Mapa, búsqueda, reservas, pagos
│       ├── dueno/         → Publicar parqueadero, gestionar reservas
│       └── admin/         → Panel de administración
```

---

## 📐 Diagramas del proyecto


### Diagrama de despliegue


&nbsp;

---

### Diagrama de contexto


&nbsp;

---

### Arquetipo de referencia (Arquitectura Hexagonal)



&nbsp;

---

### Arquitectura de referencia (5 capas)


&nbsp;

---

### Diagrama de paquetes — Backend


&nbsp;

---

### Diagrama de paquetes — Frontend Angular


&nbsp;

---

### Diagrama de clases


&nbsp;

---

### Diagrama de casos de uso


&nbsp;

---

### Diagrama de secuencia — Reservar parqueadero


&nbsp;

---

### Plataforma tecnológica


&nbsp;

---

## 🚀 Cómo correr el proyecto

### Requisitos previos

- Java 21 (OpenJDK)
- PostgreSQL 17 corriendo en el puerto `5432`
- Redis corriendo en el puerto `6379`
- Eclipse IDE + Spring Tools 4 (para el backend)
- Node.js 20+ y Angular CLI 17+ (para el frontend)

### Levantar el backend

```bash
# Iniciar servicios en Windows
net start postgresql-x64-17
net start Redis

# Opción 1 — Desde Eclipse:
# Boot Dashboard → seleccionar easy-park → Start

# Opción 2 — Con Maven desde la terminal:
mvn spring-boot:run
```

El backend queda disponible en `http://localhost:8080`

### Levantar el frontend

```bash
cd easy-park-frontend
npm install
ng serve
```

El frontend queda disponible en `http://localhost:4200`

### Configuración (`application.yml`)

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
    lock-timeout: 300      # Bloqueo Redis en segundos (5 minutos)
```


---

##  Repositorio de documentación
<img width="891" height="441" alt="Diagrama sin título-BACK END drawio" src="https://github.com/user-attachments/assets/c3bbe29f-fc8e-4492-913f-d575b54b4592" /><img width="1181" height="661" alt="arqueotipo de referencia drawio" src="https://github.com/user-attachments/assets/ee5d648c-5a20-43b8-900a-ae7e6bf5bb1f" />
<img width="751" height="881" alt="front end paquetes drawio" src="https://github.com/user-attachments/assets/a13034ae-2f77-4ab3-85b0-6bbea51d9990" />
[markdown_frontend_easypark.md](https://github.com/user-attachments/files/28040788/markdown_frontend_easypark.md)
<img width="1127" height="951" alt="DIAGRAMA DE DESPLIEGUE drawio" src="https://github.com/user-attachments/assets/24781676-1075-4a49-8e05-5c5bc8fa906c" />
<img width="1260" height="581" alt="diagrama de capas back end drawio" src="https://github.com/user-attachments/assets/1c05257a-a14d-4c11-9091-d6f1e54febde" />
<img width="791" height="811" alt="diagrama paquetes backend (1) drawio" src="https://github.com/user-attachments/assets/8d983df4-4811-42b9-bd65-0cb4b7281376" />




