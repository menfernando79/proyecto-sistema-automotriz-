# Sistema Taller Automotriz Mendoza & Asoc.

Sistema web para la gestión de un **taller mecánico**: recepción vehicular, inspección técnica,
diagnóstico, presupuesto, repuestos, cobro y trazabilidad.

![Stack](https://img.shields.io/badge/Node-Express-5ea?logo=nodedotjs) ![React](https://img.shields.io/badge/React-18-61dafb?logo=react) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-3NF-4169e1?logo=postgresql)

## Módulos

| Módulo | Qué resuelve |
|---|---|
| **Recepción** | Ingreso del vehículo con autocompletado, kilometraje, motivo y fecha. |
| **Inspección técnica** | Diagrama de corte lateral con 11 sistemas mecánicos marcables (motor, transmisión, suspensión, frenos, escape…) + checklist por sistemas. |
| **Diagnóstico** | El mecánico documenta el diagnóstico y avanza el estado de la orden. |
| **Presupuesto** | Líneas de mano de obra y repuestos; el total **se calcula**, no se tipea. Aprobación del cliente registrada. |
| **Repuestos** | Inventario con stock, stock mínimo y movimientos de entrada/salida auditados. No permite descuadrar stock. |
| **Cobro** | Registro de pago que lleva la orden a `PAGADO`, con bloqueo de doble cobro. |
| **Verificación por QR** | La orden tiene un token UUID; el cliente escanea y ve el estado sin iniciar sesión. |
| **Auditoría** | Timeline de quién hizo qué en cada orden. |
| **Reportes** | Ingresos por mes, servicios más vendidos, productividad por mecánico, consumo de repuestos. Exportación a CSV. |
| **Usuarios** | Alta, edición y baja de personal; asignación de mecánico a la orden. |

## Stack

| Capa | Tecnología |
|---|---|
| Backend | Node.js + Express (ESM), JWT, 4 roles, helmet, rate limiting |
| Frontend | React 18 + Vite, Bootstrap 5, gráficos y 3D en CSS puro |
| Datos | PostgreSQL 3NF con migraciones versionadas — modo demo en memoria si no hay BD |
| Tests | Vitest + Supertest (35 pruebas de API) |

## Puesta en marcha

**Requisitos:** Node.js 18+ (probado en v24). PostgreSQL es **opcional**.

```powershell
# 1. Backend
cd "D:\sistema taller automotriz mendoza\backend"
copy .env.example .env       # generar un JWT_SECRET: node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
npm install
npm run dev                  # http://localhost:4000

# 2. Frontend (en otra terminal)
cd "D:\sistema taller automotriz mendoza\frontend"
npm install
npm run dev                  # http://localhost:5173
```

Sin `POSTGRES_URL`, el sistema arranca en **modo demo** con datos en memoria.

## Base de datos

```powershell
$env:POSTGRES_URL = "postgresql://usuario:password@localhost:5432/taller_mendoza"
npm run migrate --prefix backend     # aplica backend/sql/migrations/ en orden
```

Las migraciones se registran en la tabla `migraciones` y cada una corre en su propia transacción,
así que nunca quedan a medio aplicar.

## Usuarios de prueba

| Usuario | Contraseña | Rol |
|---|---|---|
| admin | admin123 | Administrador |
| recepcion | rece123 | Recepcionista |
| mecanico | meca123 | Mecánico |
| cliente | cliente123 | Cliente (acceso externo, 403 al back-office) |

## Seguridad implementada

- `JWT_SECRET` obligatorio: el servidor no arranca sin él.
- Passwords con **bcrypt**; el hash nunca sale por la API.
- **Helmet** y cabeceras de seguridad HTTP.
- **Rate limiting**: 300 req/min en la API, 10/min en el login.
- **CORS** por lista blanca de orígenes.
- Autorización por rol en cada endpoint; el `CLIENTE` recibe **403**.
- QR público por **token UUID**, no por ID secuencial (evita enumerar órdenes).
- Paginación acotada y validación de entrada con `lib/validar.js`.
- El HTML de impresión fuerza tema claro para que el comprobante sea legible.

## Verificación

```powershell
npm test  --prefix backend     # 35 pruebas de API
npm run lint --prefix backend
npm run lint --prefix frontend
npm run build --prefix frontend
```

## Estructura

```
backend/
├── sql/migrations/   migraciones versionadas (001…004)
├── scripts/          corredor de migraciones, limpieza de datos de prueba
├── src/routes/       auth, recepciones, clientes, vehiculos,
│                     servicios, repuestos, usuarios, reportes, dashboard
├── src/lib/          validar.js (validación y CSV)
├── src/pg.js         capa de datos PostgreSQL
├── src/store.js      capa de datos en memoria (demo), misma interfaz
└── test/             api.test.js

frontend/src/
├── permisos.js       usePermisos(user) — única fuente de permisos de UI
├── components/       Layout, ProtectedRoute, ErrorBoundary, TransicionPagina,
│                     ContadorAnimado, Vehiculo3D, DiagramaVehiculo, Autocomplete,
│                     Paginador, BuscadorGlobal, Notificaciones, toast
├── pages/            Login, Dashboard, Recepciones, NuevaRecepcion,
│                     RecepcionDetalle, Clientes, Vehiculos, Repuestos,
│                     Reportes, Usuarios, VerificacionPublica
└── index.css         design system: tokens, tema Dark Tech, 3D, animaciones, print
```

## Documentación para IA asistente

`AGENTS.md` en la raíz documenta la arquitectura, las convenciones y las reglas de trabajo
para cualquier agente que colabore en el proyecto.
