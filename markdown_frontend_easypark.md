# 🅿️ Easy Park — Frontend Angular

### Interfaz de Usuario del Sistema de Parqueaderos Privados

**Framework:** Angular 17+  
**Universidad:** Universidad Católica del Oriente  
**Asignatura:** Arquitectura de Software  
**Año:** 2026

---

## Índice

- [Descripción](#descripción)
- [Tecnologías Utilizadas](#tecnologías-utilizadas)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Módulos y Componentes](#módulos-y-componentes)
- [Servicios](#servicios)
- [Guards e Interceptores](#guards-e-interceptores)
- [Modelos](#modelos)
- [Vistas por Rol](#vistas-por-rol)
- [Comunicación con el Backend](#comunicación-con-el-backend)
- [Cómo Correr el Proyecto](#cómo-correr-el-proyecto)
- [Variables de Entorno](#variables-de-entorno)

---

## Descripción

El frontend de **Easy Park** es una **Single Page Application (SPA)** desarrollada con **Angular 17**. Provee la interfaz de usuario para los tres roles del sistema: Conductor, Dueño de parqueadero y Administrador. Se comunica con el backend Spring Boot mediante **HTTP REST** para operaciones CRUD y **WebSocket** para actualizaciones de disponibilidad en tiempo real.

---

## Tecnologías Utilizadas

| Tecnología | Versión | Uso |
|---|---|---|
| **Angular** | 17+ | Framework principal SPA |
| **TypeScript** | 5.x | Lenguaje base del frontend |
| **Angular Router** | 17+ | Navegación entre vistas y guards |
| **HttpClient** | 17+ | Comunicación HTTP con el backend |
| **RxJS** | 7+ | Programación reactiva y observables |
| **Leaflet / Google Maps** | Latest | Mapa interactivo de parqueaderos |
| **Angular WebSocket** | 17+ | Disponibilidad en tiempo real |
| **Tailwind CSS** | 3+ | Estilos y diseño responsive |
| **JWT Decode** | 3+ | Decodificación de tokens JWT |

---

## Estructura del Proyecto

```
easy-park-frontend/
├── package.json
├── angular.json
├── tailwind.config.js
├── tsconfig.json
└── src/
    ├── main.ts                          ← bootstrapApplication(AppComponent)
    ├── app.config.ts                    ← provideRouter · provideHttpClient · JWT interceptor
    ├── app.routes.ts                    ← rutas principales del sistema
    └── app/
        ├── core/
        │   ├── services/
        │   │   ├── auth.service.ts      ← login · registro · logout · token
        │   │   ├── parking.service.ts   ← CRUD parqueaderos · búsqueda · aprobación
        │   │   ├── reserva.service.ts   ← crear · confirmar · cancelar reservas
        │   │   └── websocket.service.ts ← suscripción disponibilidad tiempo real
        │   └── guards/
        │       ├── auth.guard.ts        ← protege rutas privadas (requiere JWT)
        │       ├── role.guard.ts        ← protege rutas por rol específico
        │       └── interceptors/
        │           └── jwt.interceptor.ts ← agrega Bearer token a cada petición HTTP
        ├── shared/
        │   ├── models/
        │   │   ├── usuario.model.ts     ← interface Usuario · enum Rol
        │   │   ├── parqueadero.model.ts ← interface Parqueadero
        │   │   └── reserva.model.ts     ← interface Reserva · enum EstadoReserva
        │   └── components/
        │       ├── navbar/
        │       │   └── navbar.component.ts
        │       ├── toast/
        │       │   └── toast.component.ts
        │       ├── loader/
        │       │   └── loader.component.ts
        │       └── map/
        │           └── map.component.ts ← componente mapa reutilizable (Leaflet)
        ├── features/
        │   ├── auth/
        │   │   ├── login/
        │   │   │   ├── login.component.ts
        │   │   │   └── login.component.html
        │   │   └── register/
        │   │       ├── register.component.ts
        │   │       └── register.component.html
        │   ├── conductor/
        │   │   ├── dashboard/
        │   │   │   ├── dashboard.component.ts
        │   │   │   └── dashboard.component.html
        │   │   ├── mapa/
        │   │   │   ├── mapa.component.ts
        │   │   │   └── mapa.component.html
        │   │   ├── parqueadero-detail/
        │   │   │   ├── parqueadero-detail.component.ts
        │   │   │   └── parqueadero-detail.component.html
        │   │   ├── reserva-modal/
        │   │   │   ├── reserva-modal.component.ts
        │   │   │   └── reserva-modal.component.html
        │   │   └── mis-reservas/
        │   │       ├── mis-reservas.component.ts
        │   │       └── mis-reservas.component.html
        │   ├── dueno/
        │   │   ├── dashboard/
        │   │   │   ├── dashboard.component.ts
        │   │   │   └── dashboard.component.html
        │   │   ├── mis-parqueaderos/
        │   │   │   ├── mis-parqueaderos.component.ts
        │   │   │   └── mis-parqueaderos.component.html
        │   │   ├── crear-parqueadero/
        │   │   │   ├── crear-parqueadero.component.ts
        │   │   │   └── crear-parqueadero.component.html
        │   │   └── reservas-recibidas/
        │   │       ├── reservas-recibidas.component.ts
        │   │       └── reservas-recibidas.component.html
        │   └── admin/
        │       ├── dashboard/
        │       │   ├── dashboard.component.ts
        │       │   └── dashboard.component.html
        │       ├── pendientes/
        │       │   ├── pendientes.component.ts
        │       │   └── pendientes.component.html
        │       ├── usuarios/
        │       │   ├── usuarios.component.ts
        │       │   └── usuarios.component.html
        │       └── aprobar/
        │           ├── aprobar.component.ts
        │           └── aprobar.component.html
        └── environments/
            ├── environment.ts           ← apiUrl · wsUrl (desarrollo)
            └── environment.prod.ts      ← apiUrl · wsUrl (producción)
```

---

## Módulos y Componentes

### Auth — `/auth`

| Componente | Ruta | Descripción |
|---|---|---|
| `LoginComponent` | `/auth/login` | Formulario de inicio de sesión. Llama a `AuthService.login()` y guarda el token JWT en `localStorage`. Redirige al dashboard según el rol |
| `RegisterComponent` | `/auth/register` | Formulario de registro con selector de rol (Conductor o Dueño). Valida contraseña segura en el frontend |

### Conductor — `/conductor`

| Componente | Ruta | Descripción |
|---|---|---|
| `DashboardComponent` | `/conductor/dashboard` | Vista principal del conductor con resumen de reservas activas |
| `MapaComponent` | `/conductor/mapa` | Mapa interactivo con pines de parqueaderos disponibles. Actualiza disponibilidad en tiempo real vía WebSocket |
| `ParqueaderoDetailComponent` | `/conductor/parqueadero/:id` | Detalle del parqueadero: precio, espacios disponibles, horarios |
| `ReservaModalComponent` | Modal | Formulario de reserva: placa, horas estimadas. Inicia el bloqueo Redis de 5 minutos y muestra el timer |
| `MisReservasComponent` | `/conductor/mis-reservas` | Historial de reservas del conductor con estado y opción de cancelar |

### Dueño — `/dueno`

| Componente | Ruta | Descripción |
|---|---|---|
| `DashboardComponent` | `/dueno/dashboard` | Estadísticas del dueño: ingresos, reservas del día, espacios activos |
| `MisParqueaderosComponent` | `/dueno/mis-parqueaderos` | Lista de parqueaderos propios con opción de activar/desactivar disponibilidad |
| `CrearParqueaderoComponent` | `/dueno/crear-parqueadero` | Formulario para registrar nuevo parqueadero (queda pendiente de aprobación del admin) |
| `ReservasRecibidasComponent` | `/dueno/reservas-recibidas` | Lista de reservas recibidas con opción de aceptar o rechazar |

### Admin — `/admin`

| Componente | Ruta | Descripción |
|---|---|---|
| `DashboardComponent` | `/admin/dashboard` | Panel general con estadísticas del sistema |
| `PendientesComponent` | `/admin/pendientes` | Lista de parqueaderos pendientes de aprobación con botón Aprobar |
| `UsuariosComponent` | `/admin/usuarios` | Lista de usuarios registrados con opción de bloquear/desbloquear |
| `AprobarComponent` | Modal | Detalle del parqueadero antes de aprobarlo |

---

## Servicios

### `AuthService` — `core/services/auth.service.ts`

```typescript
login(email: string, password: string): Observable<LoginResponse>
register(data: RegisterRequest): Observable<any>
logout(): void
getToken(): string | null
getRol(): string | null
isAuthenticated(): boolean
```

### `ParkingService` — `core/services/parking.service.ts`

```typescript
getDisponibles(): Observable<Parqueadero[]>
getById(id: number): Observable<Parqueadero>
crear(data: Parqueadero): Observable<Parqueadero>
getMisParqueaderos(): Observable<Parqueadero[]>
toggleDisponibilidad(id: number, disponible: boolean): Observable<any>
aprobar(id: number): Observable<any>
getPendientes(): Observable<Parqueadero[]>
```

### `ReservaService` — `core/services/reserva.service.ts`

```typescript
iniciarReserva(parqueaderoId: number, horas: number, placa: string): Observable<Reserva>
confirmarReserva(reservaId: number, pagoId: string): Observable<Reserva>
cancelarReserva(reservaId: number): Observable<any>
getMisReservas(): Observable<Reserva[]>
getReservasParqueadero(id: number): Observable<Reserva[]>
```

### `WebSocketService` — `core/services/websocket.service.ts`

```typescript
conectar(): void
suscribirParqueadero(parqueaderoId: number): void
onDisponibilidadChange(): Observable<DisponibilidadMessage>
desconectar(): void
```

---

## Guards e Interceptores

### `AuthGuard` — `core/guards/auth.guard.ts`
Protege todas las rutas privadas. Si el usuario no tiene token JWT válido, redirige a `/auth/login`.

```typescript
canActivate(): boolean {
  if (this.authService.isAuthenticated()) return true;
  this.router.navigate(['/auth/login']);
  return false;
}
```

### `RoleGuard` — `core/guards/role.guard.ts`
Protege rutas por rol específico. Si el usuario no tiene el rol requerido, redirige a su dashboard correspondiente.

```typescript
// Uso en rutas:
{ path: 'admin', canActivate: [RoleGuard], data: { roles: ['ADMINISTRADOR'] } }
{ path: 'dueno', canActivate: [RoleGuard], data: { roles: ['DUENO'] } }
```

### `JwtInterceptor` — `core/guards/interceptors/jwt.interceptor.ts`
Agrega automáticamente el token JWT al header de cada petición HTTP saliente.

```typescript
// Agrega a cada petición:
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

---

## Modelos

### `usuario.model.ts`

```typescript
export interface Usuario {
  id: number;
  nombre: string;
  apellido: string;
  email: string;
  telefono: string;
  rol: Rol;
  activo: boolean;
}

export enum Rol {
  CONDUCTOR = 'CONDUCTOR',
  DUENO = 'DUENO',
  ADMINISTRADOR = 'ADMINISTRADOR'
}
```

### `parqueadero.model.ts`

```typescript
export interface Parqueadero {
  id: number;
  nombre: string;
  descripcion: string;
  direccion: string;
  precioHora: number;
  disponible: boolean;
  aprobado: boolean;
  capacidadTotal: number;
  espaciosDisponibles: number;
  propietarioId: number;
}
```

### `reserva.model.ts`

```typescript
export interface Reserva {
  id: number;
  conductorId: number;
  parqueaderoId: number;
  fechaInicio: string;
  horasEstimadas: number;
  totalPagar: number;
  estado: EstadoReserva;
  placaVehiculo: string;
}

export enum EstadoReserva {
  PENDIENTE_PAGO = 'PENDIENTE_PAGO',
  CONFIRMADA = 'CONFIRMADA',
  ACTIVA = 'ACTIVA',
  COMPLETADA = 'COMPLETADA',
  CANCELADA = 'CANCELADA'
}
```

---

## Vistas por Rol

### 🚗 Conductor
1. Abre la app → pantalla de login
2. Inicia sesión → redirige automáticamente a `/conductor/mapa`
3. Ve el mapa con pines de parqueaderos disponibles
4. Hace click en un pin → detalle del parqueadero
5. Click en **Reservar** → modal con formulario (placa, horas)
6. Confirma → Redis bloquea 5 minutos → aparece timer en pantalla
7. Paga con Mercado Pago → recibe notificación de confirmación
8. Puede ver sus reservas en `/conductor/mis-reservas`

### 🏠 Dueño
1. Inicia sesión → redirige a `/dueno/dashboard`
2. Ve estadísticas de sus parqueaderos
3. Puede crear nuevo parqueadero en `/dueno/crear-parqueadero`
4. Recibe notificaciones de nuevas reservas
5. Acepta o rechaza reservas desde `/dueno/reservas-recibidas`
6. Activa/desactiva disponibilidad desde `/dueno/mis-parqueaderos`

### 👮 Administrador
1. Inicia sesión → redirige a `/admin/dashboard`
2. Ve parqueaderos pendientes en `/admin/pendientes`
3. Aprueba parqueaderos para que aparezcan en el mapa
4. Gestiona usuarios desde `/admin/usuarios`

---

## Comunicación con el Backend

### HTTP REST

| Servicio | Método | Endpoint |
|---|---|---|
| Login | POST | `http://localhost:8080/api/auth/login` |
| Registro | POST | `http://localhost:8080/api/auth/register` |
| Parqueaderos disponibles | GET | `http://localhost:8080/api/parqueaderos` |
| Detalle parqueadero | GET | `http://localhost:8080/api/parqueaderos/{id}` |
| Crear parqueadero | POST | `http://localhost:8080/api/parqueaderos` |
| Aprobar parqueadero | PATCH | `http://localhost:8080/api/parqueaderos/admin/{id}/aprobar` |
| Pendientes de aprobación | GET | `http://localhost:8080/api/parqueaderos/admin/pendientes` |

### WebSocket (tiempo real)

```typescript
// Conectar
const ws = new WebSocket('ws://localhost:8080/ws/parqueaderos');

// Suscribirse a un parqueadero
ws.send(JSON.stringify({ accion: 'SUSCRIBIR', parqueaderoId: 1 }));

// Mensaje que recibe el frontend
// { "tipo": "DISPONIBILIDAD", "parqueaderoId": 1, "espaciosDisponibles": 2 }
```

---

## Cómo Correr el Proyecto

### Requisitos

- Node.js 20+
- Angular CLI 17+
- Backend Easy Park corriendo en `http://localhost:8080`

### Instalación

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/easy-park-frontend.git
cd easy-park-frontend

# Instalar dependencias
npm install

# Correr en modo desarrollo
ng serve

# Abrir en el navegador
# http://localhost:4200
```

### Crear la estructura de carpetas (primera vez)

```bash
mkdir src/app/core/services
mkdir src/app/core/guards
mkdir src/app/core/guards/interceptors
mkdir src/app/shared/models
mkdir src/app/shared/components
mkdir src/app/features/auth/login
mkdir src/app/features/auth/register
mkdir src/app/features/conductor/dashboard
mkdir src/app/features/conductor/mapa
mkdir src/app/features/conductor/parqueadero-detail
mkdir src/app/features/conductor/reserva-modal
mkdir src/app/features/conductor/mis-reservas
mkdir src/app/features/dueno/dashboard
mkdir src/app/features/dueno/mis-parqueaderos
mkdir src/app/features/dueno/crear-parqueadero
mkdir src/app/features/dueno/reservas-recibidas
mkdir src/app/features/admin/dashboard
mkdir src/app/features/admin/pendientes
mkdir src/app/features/admin/usuarios
mkdir src/app/features/admin/aprobar
```

### Build para producción

```bash
ng build --configuration production
# Genera la carpeta dist/ lista para desplegar en Railway
```

---

## Variables de Entorno

### `src/environments/environment.ts` (desarrollo)

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080',
  wsUrl: 'ws://localhost:8080/ws/parqueaderos'
};
```

### `src/environments/environment.prod.ts` (producción)

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://easy-park-backend.railway.app',
  wsUrl: 'wss://easy-park-backend.railway.app/ws/parqueaderos'
};
```

---

> **Easy Park Frontend** — Angular SPA  
> Universidad Católica del Oriente · 2026
