# 🅿️ Easy Park
### Sistema de Parqueaderos Privados en Tiempo Real

**Autores:** Diego Alejandro Vega Vanegas
**Universidad:** Universidad catolica del Oriente  
**Asignatura:** Arquitectura de Software  
**Año:** 2026

---

## Índice

- [Descripción del Proyecto](#descripción-del-proyecto)
- [Contexto](#contexto)
- [Roles del Sistema](#roles-del-sistema)
- [Requisitos Funcionales](#requisitos-funcionales)
- [Requisitos No Funcionales](#requisitos-no-funcionales)
- [Funcionalidades Críticas](#funcionalidades-críticas)
- [Atributos de Calidad](#atributos-de-calidad)
- [Restricciones Técnicas](#restricciones-técnicas)
- [Restricciones de Negocio](#restricciones-de-negocio)
- [Herramientas y Tecnologías Seleccionadas](#herramientas-y-tecnologías-seleccionadas)
- [Arquitectura del Sistema](#arquitectura-del-sistema)
- [Cómo Correr el Proyecto](#cómo-correr-el-proyecto)

---

## Descripción del Proyecto

**Easy Park** es una plataforma web reactiva que conecta conductores con dueños de parqueaderos privados (garajes de casas u otros espacios privados) en una ciudad. Permite buscar espacios disponibles en tiempo real, reservarlos con bloqueo temporal vía Redis, procesar pagos de forma segura y recibir notificaciones automáticas.

---

## Contexto

Los dueños de parqueaderos o espacios privados tendrán la oportunidad de ganar dinero alquilando celdas cómodas y bien ubicadas para guardar vehículos (carros y motos). Muchos conductores se enfrentan frecuentemente al problema de no saber dónde dejar su vehículo durante las vacaciones o en las noches. Los propietarios de estos lugares podrán brindar en calidad de alquiler espacios propios para generar ingresos adicionales y a su vez ofrecer una alternativa a personas que lo necesitan.

---

## Roles del Sistema

| Rol | Descripción |
|---|---|
| 🚗 **Conductor** | Busca parqueaderos disponibles, realiza reservas y efectúa pagos |
| 🏠 **Dueño del parqueadero** | Publica, edita y gestiona sus espacios de parqueo y reservas recibidas |
| 👮 **Administrador** | Aprueba parqueaderos, gestiona usuarios y monitorea el sistema |

---

## Requisitos Funcionales

### CU001 — Crear Administrador
- **RF1** El administrador debe iniciar sesión en el sistema
- **RF2** Formulario con campos obligatorios: nombre, apellido, tipo y número de identificación, correo, contraseña
- **RF3** Validar correo único y contraseña segura (mínimo 8 caracteres, mayúscula, número, carácter especial)
- **RF4** Si la validación es exitosa, crear la cuenta
- **RF5** Confirmar la creación exitosa

### CU002 — Iniciar Sesión
- **RF1** Proporcionar formulario de inicio de sesión
- **RF2** Validar las credenciales ingresadas
- **RF3** Si son válidas, redirigir a la página principal
- **RF4** Si son inválidas, mostrar mensaje de error

### CU003 — Registro de Usuario
- **RF1** El usuario puede iniciar sesión con nombre y contraseña
- **RF2** El usuario puede crear cuenta con nombre, apellido, contraseña, tipo de vehículo, placa, celular y correo

### CU004 — Olvidar Contraseña
- **RF1** Formulario para ingresar correo o nombre de usuario
- **RF2** Enviar token de restablecimiento al correo
- **RF3** Permitir restablecer contraseña con el token
- **RF4** Validar el token antes de permitir el restablecimiento

### CU009 — Búsqueda *(Reto Técnico)*
- **RF1** Buscar parqueaderos disponibles por GPS (radio de 1 km automático)
- **RF2** Filtrar por ubicación, horario y tipo de vehículo
- **RF3** Mostrar en tiempo real las celdas disponibles del parqueadero seleccionado

### CU010 — Reserva *(Reto Técnico)*
- **RF1** Completar formulario: placa, tipo de vehículo, hora de entrada y salida estimada
- **RF2** Validar disponibilidad de la celda en el horario solicitado
- **RF3** Asignar celda y generar comprobante digital con código QR
- **RF4** Enviar notificación automática 1 hora antes del inicio
- **RF5** Enviar segunda notificación 15 minutos antes del fin

### CU011 — Gestión de Reserva *(Reto Técnico)*
- **RF1** El dueño puede aceptar la reserva
- **RF2** El dueño puede rechazar la reserva
- **RF3** Si el usuario no paga en el tiempo límite, el espacio se libera automáticamente
- **RF4** El sistema envía notificación al usuario con el estado de su reserva

### CU012 — Ofrecer Parqueadero
- **RF1** Registrar parqueadero: dirección, horarios, tipos de vehículos, celdas, tarifa por hora
- **RF2** Validar dirección no duplicada y horarios coherentes
- **RF3** El administrador debe aprobar antes de que sea visible
- **RF4** El propietario puede editar o desactivar en cualquier momento

### CU013 — Pagos *(Crítico)*
- **RF1** Pago seguro mediante pasarela de pagos (web o móvil, antes de usar el parqueadero)

### CU014 — Gestión de Facturación
- **RF1** El usuario recibe factura con código QR para verificar su identidad al llegar

### CU015 — Notificación
- **RF1** El dueño recibe notificación cuando su espacio es reservado
- **RF2** El sistema bloquea el espacio reservado para nuevas reservas

### CU016 — Gestión de Multas
- **RF1** Detectar automáticamente si un vehículo excede el tiempo de reserva
- **RF2** Calcular multa proporcional al tiempo excedido
- **RF3** Notificar al usuario con el monto y plazo de pago
- **RF4** Bloquear nuevas reservas hasta que la multa sea pagada
- **RF5** El administrador puede revisar y modificar multas en casos excepcionales

### CU017 — Pago de Reserva/Multa
- **RF1** Pagar mediante tarjeta, billetera virtual o transferencia bancaria
- **RF2** Generar recibo digital automáticamente tras confirmar el pago
- **RF3** En caso de pago fallido, notificar y permitir reintentar

---

## Requisitos No Funcionales

### Rendimiento y Escalabilidad
- Responder en menos de **5 segundos** para búsqueda de parqueaderos
- Escalable para admitir nuevos lugares dentro del mapa

### Usabilidad
- Interfaz intuitiva y fácil de usar
- Soporte de traducción en diferentes idiomas
- Optimizada para móviles y computadores

### Seguridad
- Autenticación con usuario y contraseña
- Funcionalidades diferenciadas por rol
- Protección de datos personales conforme a normativas

### Disponibilidad y Fiabilidad
- Disponibilidad mínima del **99%** del tiempo
- Copias de seguridad automáticas de la base de datos

---

## Funcionalidades Críticas

| Identificador | Historia de Usuario | PoC |
|---|---|---|
| **CU-009-RF1** | Buscar parqueaderos por GPS en radio de 1 km | ✅ Sí |
| **CU-010-RF1** | Registrar placa, tipo de vehículo y periodo de entrada/salida | No |
| **CU-010-RF3** | Asignar celda y generar comprobante con código QR | ✅ Sí |
| **CU-011-RF3** | Liberar espacio automáticamente si el usuario no paga a tiempo | ✅ Sí — requiere jobs/watchers |
| **CU-011-RF4** | Notificar al usuario el estado de su reserva | ✅ Sí |
| **CU-013-RF1** | Pago seguro mediante pasarela (Mercado Pago) | ✅ Sí — requiere integración API |
| **CU-014-RF1** | Factura con código QR para verificar identidad | ✅ Sí |

---

## Atributos de Calidad

### Priorización (Mapa de Empatía — resultado ponderado)

| # | Atributo de Calidad | % Ponderado |
|---|---|---|
| 1 | **Seguridad** | 10.6% |
| 2 | **Rendimiento** | 10.3% |
| 3 | **Usabilidad** | 9.9% |
| 4 | **Disponibilidad** | 9.9% |
| 5 | **Costo** | 8.4% |
| 6 | **Escalabilidad** | 7.7% |
| 7 | **Confiabilidad** | 7.7% |
| 8 | **Capacidad para ser soportado** | 7.7% |
| 9 | **Capacidad para ser administrado** | 7.0% |
| 10 | **Capacidad para ser mantenido** | 5.9% |
| 11 | **Interoperabilidad** | 5.5% |
| 12 | **Internacionalización** | 5.5% |
| 13 | **Capacidad para ser desplegado** | 4.0% |

### Escenarios de Calidad Priorizados (Top 10)

| Código | Atributo | Escenario |
|---|---|---|
| ESC-CAL-SEG-001 | Seguridad | Ingreso al sistema de forma exitosa |
| ESC-CAL-SEG-002 | Seguridad | Intento fallido por contraseña incorrecta |
| ESC-CAL-SEG-0022 | Seguridad | Propietario accede solo a sus funciones |
| ESC-CAL-SEG-0023 | Seguridad | Administrador accede a gestión global |
| ESC-CAL-SEG-0031 | Seguridad | Inicio de sesión con Gmail/Facebook/Office365 |
| ESC-CAL-SEG-0041 | Seguridad | Datos de tarjeta cifrados en pagos |
| ESC-CA-USA-0001 | Usabilidad | Conductor reserva en menos de 3 pasos |
| ESC-CA-USA-0022 | Usabilidad | Mensaje claro si datos son inválidos |
| ESC-CA-USA-0023 | Usabilidad | Confirmación visible al publicar parqueadero |
| ESC-CA-USA-0033 | Usabilidad | Formulario adaptable en móvil sin desplazamientos excesivos |

---

## Restricciones Técnicas

| Tipo | Restricción | Justificación |
|---|---|---|
| **Tecnología base** | Accesible desde navegador web sin instalación | Acceso rápido sin complicaciones |
| **Tecnología base** | Adaptable a móviles y computadores | La mayoría usará el celular |
| **Compatibilidad** | Compatible con Chrome, Edge y Firefox | Acceso sin problemas técnicos |
| **Concurrencia** | Soportar múltiples usuarios simultáneos | Alta concurrencia en horas pico |
| **Tiempo real** | Disponibilidad actualizada en tiempo real | Evitar doble reserva |
| **Disponibilidad** | Sistema disponible casi 24/7 | Usuarios pueden necesitar parqueadero en cualquier momento |
| **Arquitectura** | Arquitectura en capas o hexagonal | Facilitar mantenimiento y crecimiento |
| **Desarrollo** | Aplicar principios SOLID | Mejorar calidad del código |
| **Diseño** | Bajo acoplamiento entre módulos | Cambios en pagos no afectan reservas |
| **Metodología** | Metodologías ágiles | Adaptarse a cambios progresivamente |

---

## Restricciones de Negocio

| Tipo | Restricción | Plan de Acción |
|---|---|---|
| **Humano** | Disponibilidad limitada por responsabilidades académicas | Dividir tareas por módulos, priorizar funcionalidades críticas |
| **Tiempo** | Calendario académico con fechas de entrega fijas | Usar Scrum, definir MVP primero, pruebas continuas |
| **Legal** | Normativas de alquiler de espacios (Colombia) | Investigar normativas locales, incluir términos y condiciones |
| **Legal** | Ley 1581 de 2012 — Habeas Data | Políticas de privacidad, cifrar contraseñas, limitar acceso por roles |
| **Legal** | Seguridad en pagos digitales (PCI-DSS) | No almacenar datos de tarjetas, generar comprobantes digitales |
| **Tecnológico** | Dependencia de GPS e internet | GPS opcional, búsqueda manual por ubicación |
| **Seguridad** | Accesos no autorizados | Validación frontend y backend, control de roles |
| **Presupuesto** | Desembolsos por hitos aprobados | Monitoreo constante, entrega de hitos pequeños |

---

## Herramientas y Tecnologías Seleccionadas

### Frontend

| Herramienta | Estado | Justificación |
|---|---|---|
| **Angular** | ✅ Seleccionado | Estructura organizada que evita errores en datos de reservas y mapas. Framework empresarial de Google |
| React.js | ❌ No seleccionado | No se eligió para priorizar robustez estructural de Angular |

### Backend

| Herramienta | Estado | Justificación |
|---|---|---|
| **Spring Boot** | ✅ Seleccionado | Arquitectura limpia y hexagonal, separación por capas, proyecto orientado a largo plazo |
| Node.js | ❌ No seleccionado | El equipo tiene mayor experiencia en Java |
| Django | ❌ No seleccionado | El equipo no tiene experiencia en Python |

### Hosting

| Herramienta | Estado | Justificación |
|---|---|---|
| **Railway** | ✅ Seleccionado | Curva de aprendizaje baja, MVP fácil de desplegar, costos accesibles |
| AWS | ❌ No seleccionado | Pago por uso costoso, curva de aprendizaje alta |
| Google Cloud / Azure | ❌ No seleccionado | Más compleja y costosa para la etapa actual |

### Base de Datos

| Herramienta | Estado | Justificación |
|---|---|---|
| **PostgreSQL 17** | ✅ Seleccionado | Open source gratuita, conocimiento del equipo, sistema relacional robusto |
| OracleSQL | ❌ No seleccionado | Costo alto con crecimiento |
| NoSQL | ❌ No seleccionado | Puede ocasionar duplicidad; la flexibilidad no es necesaria |

### Otras Tecnologías

| Tecnología | Estado | Función |
|---|---|---|
| **Redis** | ✅ Seleccionado | Caché y bloqueo de espacios en tiempo real (mutex 5 min) |
| **Spring Security + JWT** | ✅ Seleccionado | Autenticación y autorización por roles |
| **GitHub Actions** | ✅ Seleccionado | CI/CD — automatiza despliegues y pruebas |
| **WebSocket API** | ✅ Seleccionado | Mapa en tiempo real bidireccional |
| **Firebase Cloud Messaging** | ✅ Seleccionado | Notificaciones push a conductores y dueños |
| **Mercado Pago** | ✅ Seleccionado | Pasarela de pagos popular en Latinoamérica |
| **Google Maps Platform** | ✅ Seleccionado | Mapas interactivos con ubicación precisa |
| REST API | ❌ No seleccionado | No permite actualización en tiempo real del mapa |
| Stripe | ❌ No seleccionado | Poco conocido en Colombia, genera inseguridad al pagar |

### Arquitecturas Aplicadas

| Arquitectura | Estado | Justificación |
|---|---|---|
| **Hexagonal** | ✅ Aplicada | Separa lógica de negocio del mundo externo con puertos y adaptadores |
| **En Capas** | ✅ Aplicada | Frontend, lógica de negocio y acceso a datos bien separados |
| **Microservicios** | ✅ Aplicada | Módulos independientes: usuarios, reservas, pagos, disponibilidad |
| **Event-Driven** | ✅ Aplicada | Reserva → pago → confirmación como cadena de eventos |
| **API-First** | ✅ Aplicada | Angular, app móvil y servicios externos se comunican de forma clara |
| **Contenedores (Docker)** | ✅ Aplicada | Consistencia entre entornos de desarrollo y producción |

---

## Arquitectura del Sistema

### Diagrama de Capas

```
┌──────────────────────────────────────────────────────┐
│             CAPA DE PRESENTACIÓN                     │
│  Angular SPA · Google Maps · WebSocket Client        │
│  Dashboard Conductor / Dueño / Admin                 │
└─────────────────────┬────────────────────────────────┘
                      │ HTTP / WebSocket
┌─────────────────────▼────────────────────────────────┐
│          CAPA DE API GATEWAY / SEGURIDAD              │
│  JwtAuthFilter · JwtService · SecurityConfig         │
│  BCrypt · CORS · Control de roles por endpoint       │
└─────────────────────┬────────────────────────────────┘
                      │
┌─────────────────────▼────────────────────────────────┐
│          CAPA DE LÓGICA DE NEGOCIO                    │
│  AuthController · ParqueaderoController              │
│  ReservaService · RedisParqueaderoService            │
│  WebSocketHandler                                    │
└─────────────────────┬────────────────────────────────┘
                      │
┌─────────────────────▼────────────────────────────────┐
│            CAPA DE ACCESO A DATOS                     │
│  UsuarioRepository · ParqueaderoRepository           │
│  ReservaRepository · RedisTemplate · JPA/Hibernate   │
└─────────────────────┬────────────────────────────────┘
                      │
┌─────────────────────▼────────────────────────────────┐
│        CAPA DE INFRAESTRUCTURA / PERSISTENCIA         │
│  PostgreSQL 17 · Redis · Firebase FCM · Mercado Pago │
└──────────────────────────────────────────────────────┘
```

### Flujo de una Reserva

```
1. Conductor inicia sesión → recibe token JWT
2. Carga mapa con parqueaderos disponibles
3. Se conecta al WebSocket → suscripción en tiempo real
4. Selecciona parqueadero → inicia reserva
5. Redis bloquea el espacio por 5 minutos (mutex atómico)
6. WebSocket notifica a todos los usuarios → mapa se actualiza
7. Conductor paga con Mercado Pago
8. Mercado Pago envía webhook → backend confirma reserva
9. PostgreSQL actualiza espacio · Redis libera bloqueo
10. Firebase FCM notifica al celular del conductor
```

---

## Cómo Correr el Proyecto

### Requisitos

- Java 21 (OpenJDK)
- PostgreSQL 17 en puerto `5432`
- Redis en puerto `6379`
- Eclipse IDE + Spring Tools 4 (backend)
- Node.js 20+ + Angular CLI 17+ (frontend)

### Backend

```bash
# Iniciar servicios (Windows)
net start postgresql-x64-17
net start Redis

# Correr desde Eclipse → Boot Dashboard → Start easy-park
# O con Maven:
mvn spring-boot:run
```

### Frontend

```bash
cd easy-park-frontend
npm install
ng serve
# Abrir http://localhost:4200
```

### Variables de Entorno (`application.yml`)

```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:postgresql://127.0.0.1:5432/easy_park
    username: postgres
    password: postgres
  data:
    redis:
      host: localhost
      port: 6379
app:
  jwt:
    secret: easyPark2024SecretKeyMuyLargaParaQueSeaSegura
    expiration: 86400000
  parking:
    lock-timeout: 300
```

### Credenciales de administrador por defecto

```
Email:    admin@easypark.co
Password: admin123
```

---

> **Easy Park** — Proyecto universitario de Arquitectura de Software  
> Universidad Cooperativa de Colombia · 2026  
> Cumplimiento Ley 1581 de 2012 — Habeas Data
