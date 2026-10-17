# Guía para agentes — Taller Automotriz Mendoza & Asoc.

Sistema web interno de recepción vehicular de un taller: órdenes de recepción, checklist
digital, diagnóstico, evidencia fotográfica, cobro por QR, verificación por QR y auditoría.

> Este archivo es leído por opencode, Claude Code, Codex, Antigravity y otros agentes.
> `CLAUDE.md` contiene lo mismo para Claude Code.

## Stack y puertos

| Capa | Tecnología | Puerto |
|---|---|---|
| Backend | Node + Express (ESM), JWT, roles | 4000 |
| Frontend | React 18 + Vite + Bootstrap 5 | 5173 |
| Base de datos | PostgreSQL | — |

## Ejecutar

```powershell
# backend
Copy-Item backend\.env.example backend\.env   # POSTGRES_URL vacío = modo demo
npm install --prefix backend
npm run dev --prefix backend

# frontend
npm install --prefix frontend
npm run dev --prefix frontend
# abrir http://localhost:5173
```

Usuarios demo: `admin/admin123`, `recepcion/rece123`, `mecanico/meca123`, `cliente/cliente123`.

## Estructura

```
backend/
  sql/migrations/   migraciones versionadas (001..004) — aplicar con npm run migrate
  scripts/          migrar.js (corredor), limpiar-pruebas.js
  src/routes/       auth, recepciones, clientes, vehiculos, servicios,
                    repuestos, usuarios, reportes, dashboard
  src/middleware/   auth.js (JWT sign/auth, requireRole)
  src/lib/          validar.js (texto, entero, decimal, esFechaISO, esEmail, paginar, aCSV)
  src/pg.js         capa de datos PostgreSQL
  src/store.js      capa de datos en memoria (demo) — misma interfaz que pg.js
  src/data.js       elige el store según POSTGRES_URL  → exporta `store`
  src/app.js        app Express (helmet, CORS, rate limit, rutas, errores) — exporta `app`
backend/test/       api.test.js (vitest + supertest, corre en modo demo)
frontend/src/
  api.js            wrapper fetch + mapa ESTADOS + descargar() para CSV
  permisos.js       usePermisos(user) — única fuente de permisos de UI
  App.jsx           rutas anidadas + AuthCtx (useAuth) + ErrorBoundary
  components/       Layout, ProtectedRoute, SinPermiso, ErrorBoundary,
                    TransicionPagina, ContadorAnimado, Vehiculo3D,
                    Autocomplete, Paginador, BuscadorGlobal, Notificaciones,
                    DiagramaVehiculo, toast.js
  pages/            Login, Dashboard, Recepciones, NuevaRecepcion, RecepcionDetalle,
                    Clientes, Vehiculos, Repuestos, Reportes, Usuarios,
                    VerificacionPublica
  index.css         design system (tokens, kpi-card, sb-*, lg-*, 3D, animaciones, print)
```

## Módulos del taller

- **Repuestos/inventario:** catálogo con stock, stock mínimo y movimientos auditados
  (`movimientos_stock`, transaccional con `SELECT … FOR UPDATE`).
- **Presupuesto:** `servicios` (mano de obra) + `servicios_repuestos`. El total se **calcula**
  (`recalcularPresupuesto`), no se tipea. Se registra la aprobación del cliente.
- **Roles:** `ADMINISTRADOR`, `RECEPCIONISTA`, `MECANICO`, `CLIENTE`. La UI usa
  `usePermisos(user)` de `frontend/src/permisos.js`; el backend manda con `requireRole(...)`.
- **Estados:** `CREADO → EN_DIAGNOSTICO → EN_REPARACION → PAGADO → ENTREGADO` (+ `CANCELADO`).
  No inventar estados sin actualizar `frontend/src/api.js` (`ESTADOS`) y `TRANSICIONES`
  en `backend/src/routes/recepciones.js`.
- **QR público:** la orden tiene `token_publico` UUID. El QR apunta a `/ver/:token`
  (`GET /api/recepciones/public/:token`). **Nunca volver a filtrar por ID secuencial.**
- **Inspección:** `DiagramaVehiculo.jsx` son 11 zonas de sistemas mecánicos. Los nombres de zona
  son la clave de match con el checklist guardado (`checklistAZonas`).

## Convenciones (importante)

- **Tema oscuro:** `data-bs-theme="dark"` es intencional. Los tokens reales están en
  `:root[data-bs-theme="dark"]` (bloque `TEMA DARK TECH` de `index.css`), no en el `:root` de
  arriba, que conserva valores legacy. No revertir.
- **Capa de datos:** las rutas usan `import { store } from '../data.js'` y siempre `await`.
  Toda operación nueva debe implementarse **igual** en `pg.js` y en `store.js`.
- **Permisos de UI:** `usePermisos(user)`. Prohibido hardcodear arrays de roles en las páginas.
- **Migraciones:** solo `npm run migrate --prefix backend`. Nunca SQL a mano.
- **Seguridad:** `JWT_SECRET` obligatorio; helmet; rate limit (300/min API, 10/min login);
  CORS por `CORS_ORIGINS`; paginación acotada por `paginar()`; errores de integridad → 409.
- **Frontend:** usar `api()` (nunca `fetch` directo). Feedback con `toast.*`; nada de `alert()`.
  Confirmar acciones destructivas. Bolivia: `es-BO`, `Bs`.
- **UI:** reutilizar `.kpi-card`/`.kpi-etiqueta`, `.filtros`/`.campo`, `.sb-*`, `.lg-*`, `.no-print`.
- **No comentar el código** salvo que se pida.

## Verificación

```powershell
npm test  --prefix backend     # 35 pruebas (auth, roles, validación, ciclo de vida, presupuesto, QR, CRUD, reportes)
npm run lint --prefix backend
npm run lint --prefix frontend  # 0 errores; warnings de exhaustive-deps conocidos
npm run build --prefix frontend
```

- Los tests corren contra el almacén en memoria (sin PostgreSQL): **no cubren `pg.js`**.
  Si tocás `pg.js`, probá contra la base real (`npm start` + curl).
- **No** matar todos los procesos `node` (mata también Vite). Detener solo el PID del puerto.
- El agente no puede ver la UI renderizada: pedir confirmación visual al usuario.

## Skills disponibles

- Globales (`~/.agents/skills`, también en `~/.claude/skills`): pack "superpowers"
  (TDD, debugging, planes, code review, subagentes) + `agent-orchestration`.
- Del proyecto: `docs/skills/frontend-taller-mendoza/SKILL.md` (frontend: tema, design system,
  permisos, 3D). Regla equivalente para Antigravity en `.agent/rules/frontend-taller.md`.
