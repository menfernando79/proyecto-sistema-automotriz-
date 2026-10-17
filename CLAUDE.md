@AGENTS.md

# Taller Mendoza & Asoc. — Claude Code

El contexto completo del proyecto (stack, estructura, convenciones, comandos y
gotchas) está en `@AGENTS.md`, que se importa arriba. Resumen operativo:

- Backend Express en el puerto 4000 (`backend/`), frontend React+Vite en el 5173 (`frontend/`).
- Capa de datos: usar `store` de `data.js` con `await`; implementar toda operación en `pg.js` **y** `store.js`.
- Feedback de UI con `toast.*` (nunca `alert`); confirmar acciones destructivas.
- Verificación: `npm run build --prefix frontend` + pruebas HTTP; no matar todos los `node`.
- No comentar el código salvo que se pida.
