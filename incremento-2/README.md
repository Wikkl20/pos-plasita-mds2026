# Incremento 2 — Módulo POS Funcional

**Semanas:** 7 a 12  
**Estado:** 🔄 En progreso

---

## Objetivo del Incremento

Construir el módulo POS con arquitectura offline-first y demostrar el ciclo completo: **venta sin conexión → sincronización automática → consolidación en servidor central**.

## Entregables

| Artefacto | Ubicación | Estado |
|-----------|-----------|--------|
| Documento de arquitectura | `docs/03-arquitectura.md` | ✅ |
| Plan de calidad ISO 25010 | `docs/04-plan-calidad.md` | ✅ |
| Esquema SQLite (caja) | `docs/03-arquitectura.md` sección 4.1 | ✅ |
| Esquema PostgreSQL (servidor) | `docs/03-arquitectura.md` sección 4.2 | ✅ |
| App Electron + React (caja) | `src/caja/` | 🔄 |
| API Node.js + Express (sync) | `src/servidor/` | 🔄 |
| Casos de prueba aceptación | `docs/05-casos-de-prueba.md` | ✅ |
| Reporte JMeter (carga) | `incremento-2/resultados-jmeter/` | ⬜ Pendiente |
| Informe final | `incremento-2/informe-final.md` | ⬜ Pendiente |

## Issues del Incremento

Ver todas las issues etiquetadas con `incremento-2` en el [tablero del proyecto →](#)

## Demostración del ciclo completo

```
[Caja 1 - offline]          [Red restaurada]       [Servidor]
Registrar 3 ventas    →     Sync automático   →    Ver en dashboard
(SQLite local)              (≤ 60 segundos)         (PostgreSQL)
```

Este ciclo de 2 minutos demuestra que la arquitectura resuelve el problema de Plasita sin importar el estado del servidor.

