# 03 · Arquitectura Técnica — Sistema POS Offline-First

**Proyecto:** Sistema POS Offline-First — Tienda Plasita  
**Versión:** 1.0 · Mayo 2026  
**Autor:** Lazo Eulogio, Emanuel Alexander (Arquitecto)

---

## 1. Decisión Arquitectónica

### 1.1 Patrón elegido: Offline-First con Sincronización Eventual

El problema de Plasita es un **problema arquitectónico**, no de funcionalidades. La solución no es reemplazar VipSystem por otro sistema centralizado: es cambiar el patrón de dependencia.

| Arquitectura anterior | Arquitectura propuesta |
|----------------------|----------------------|
| Todas las cajas dependen del servidor | Cada caja opera de forma autónoma |
| Sin servidor → sin ventas | Sin servidor → ventas continúan |
| Punto único de fallo | Sin punto único de fallo |
| Sync en tiempo real (obligatoria) | Sync eventual (cuando hay conexión) |

### 1.2 Referentes del patrón

Este patrón es usado por sistemas POS modernos en todo el mundo: Square POS, Toast POS, Shopify POS. Todos operan offline-first porque la conectividad en entornos comerciales nunca es 100% garantizada.

---

## 2. Diagrama de Arquitectura

```
╔══════════════════════════════════════════════════════════════════╗
║                    TIENDA PLASITA                                ║
║                                                                  ║
║  ┌─────────────┐  ┌─────────────┐       ┌─────────────┐         ║
║  │   CAJA 1    │  │   CAJA 2    │  ...  │   CAJA 14   │         ║
║  │             │  │             │       │             │         ║
║  │ Electron    │  │ Electron    │       │ Electron    │         ║
║  │ + React     │  │ + React     │       │ + React     │         ║
║  │             │  │             │       │             │         ║
║  │ ┌─────────┐ │  │ ┌─────────┐ │       │ ┌─────────┐ │         ║
║  │ │ SQLite  │ │  │ │ SQLite  │ │       │ │ SQLite  │ │         ║
║  │ │ local   │ │  │ │ local   │ │       │ │ local   │ │         ║
║  │ └─────────┘ │  │ └─────────┘ │       │ └─────────┘ │         ║
║  │ Cola sync   │  │ Cola sync   │       │ Cola sync   │         ║
║  └──────┬──────┘  └──────┬──────┘       └──────┬──────┘         ║
║         │                │                     │                ║
║         └────────────────┼─────────────────────┘                ║
║                          │ (cuando hay conexión)                ║
║                          ▼                                       ║
║              ┌───────────────────────┐                          ║
║              │   SERVIDOR SYNC       │                          ║
║              │   Node.js + Express   │                          ║
║              │   POST /sync          │                          ║
║              │   GET  /ventas        │                          ║
║              └───────────┬───────────┘                          ║
║                          │                                       ║
║                          ▼                                       ║
║              ┌───────────────────────┐                          ║
║              │  PC ADMINISTRADORA    │                          ║
║              │  PostgreSQL           │                          ║
║              │  Dashboard web        │                          ║
║              └───────────────────────┘                          ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## 3. Stack Tecnológico

### 3.1 Capa 1 — Aplicación de Caja

| Componente | Tecnología | Versión | Justificación |
|-----------|-----------|---------|---------------|
| Framework desktop | **Electron** | 28.x | Empaqueta apps web como ejecutable .exe para Windows. Sin aprender lenguaje nuevo. |
| Interfaz de usuario | **React** | 18.x | Biblioteca de UI en JavaScript. El equipo ya lo conoce. |
| Base de datos local | **SQLite** | 3.x | Embebida en el ejecutable. Sin instalación separada. Usada por WhatsApp, Firefox, Android. |
| Estilos | **Tailwind CSS** | 3.x | Utility-first, rápido de implementar |

**¿Por qué Electron y no una app web?**  
Las cajas de Plasita son PCs Windows. Electron permite distribuir la app como un instalador `.exe` que cualquier técnico puede instalar. Una app web requeriría un servidor local adicional y el navegador siempre abierto, lo que es menos robusto para un entorno de caja 8 horas diarias.

### 3.2 Capa 2 — Servidor de Sincronización

| Componente | Tecnología | Versión | Justificación |
|-----------|-----------|---------|---------------|
| Runtime | **Node.js** | 20 LTS | Mismo ecosistema JavaScript del equipo |
| Framework API | **Express** | 4.x | Minimalista, suficiente para los 3 endpoints necesarios |
| ORM | **Prisma** | 5.x | Genera queries tipados para PostgreSQL, evita SQL manual propenso a errores |

### 3.3 Capa 3 — Base de Datos Central

| Componente | Tecnología | Versión | Justificación |
|-----------|-----------|---------|---------------|
| Base de datos | **PostgreSQL** | 16.x | Open source, robusto, gratuito. Estándar de industria. |
| Hosting | PC Administradora | — | Ya existe en Plasita. Sin costo adicional de infraestructura. |

### 3.4 Herramientas de Desarrollo y Calidad

| Herramienta | Uso | Costo |
|------------|-----|-------|
| GitHub | Control de versiones + gestión del proyecto | Gratis |
| GitHub Actions | CI/CD: ejecuta tests en cada push | Gratis |
| SonarCloud | Análisis estático de código (OWASP Top 10) | Gratis (open source) |
| Apache JMeter | Pruebas de carga (14 usuarios simultáneos) | Gratis |
| Figma | Wireframes y prototipos UI | Gratis |
| Jest | Tests unitarios del motor de sync | Gratis |

**Costo total del stack: S/ 0.00**

---

## 4. Diseño de Base de Datos

### 4.1 SQLite — Esquema por Caja

```sql
-- Tabla principal de ventas
CREATE TABLE ventas (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    fecha       DATETIME DEFAULT CURRENT_TIMESTAMP,
    cajero_id   INTEGER NOT NULL,
    total       DECIMAL(10,2) NOT NULL,
    metodo_pago TEXT CHECK(metodo_pago IN ('efectivo','yape','plin')),
    sync_estado TEXT DEFAULT 'pendiente' CHECK(sync_estado IN ('pendiente','sincronizado'))
);

-- Detalle de cada venta (productos)
CREATE TABLE detalle_venta (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    venta_id    INTEGER REFERENCES ventas(id),
    producto_id INTEGER NOT NULL,
    cantidad    INTEGER NOT NULL,
    precio_unit DECIMAL(10,2) NOT NULL,
    subtotal    DECIMAL(10,2) NOT NULL
);

-- Cola de sincronización
CREATE TABLE cola_sync (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    venta_id    INTEGER REFERENCES ventas(id),
    intentos    INTEGER DEFAULT 0,
    ultimo_intento DATETIME,
    estado      TEXT DEFAULT 'pendiente'
);

-- Catálogo local de productos (se actualiza desde el servidor)
CREATE TABLE productos (
    id          INTEGER PRIMARY KEY,
    codigo      TEXT UNIQUE NOT NULL,
    nombre      TEXT NOT NULL,
    precio      DECIMAL(10,2) NOT NULL,
    stock_local INTEGER DEFAULT 0,
    stock_min   INTEGER DEFAULT 5
);
```

### 4.2 PostgreSQL — Esquema Central

```sql
-- Registro consolidado de todas las cajas
CREATE TABLE ventas_consolidadas (
    id          SERIAL PRIMARY KEY,
    caja_id     INTEGER NOT NULL,
    venta_local_id INTEGER NOT NULL,
    fecha       TIMESTAMP NOT NULL,
    cajero_id   INTEGER NOT NULL,
    total       DECIMAL(10,2) NOT NULL,
    metodo_pago TEXT NOT NULL,
    sincronizado_en TIMESTAMP DEFAULT NOW(),
    UNIQUE(caja_id, venta_local_id)  -- evita duplicados en re-sync
);

-- Inventario consolidado
CREATE TABLE inventario (
    producto_id INTEGER PRIMARY KEY,
    codigo      TEXT UNIQUE NOT NULL,
    nombre      TEXT NOT NULL,
    precio      DECIMAL(10,2) NOT NULL,
    stock_total INTEGER DEFAULT 0,
    stock_min   INTEGER DEFAULT 10,
    actualizado_en TIMESTAMP DEFAULT NOW()
);
```

---

## 5. Motor de Sincronización

### 5.1 Flujo de sincronización

```
┌─────────────────────────────────────────────┐
│           MOTOR DE SYNC (cada 30 seg)        │
│                                             │
│  1. ¿Hay ventas en cola_sync con estado     │
│     'pendiente'?                            │
│         NO → esperar 30 seg                 │
│         SÍ → continuar                      │
│                                             │
│  2. ¿Hay conexión al servidor?              │
│         NO → registrar intento fallido,     │
│              esperar 30 seg                 │
│         SÍ → continuar                      │
│                                             │
│  3. Agrupar ventas pendientes en un lote    │
│     JSON y enviar a POST /sync              │
│                                             │
│  4. ¿Servidor responde 200 OK?              │
│         NO → incrementar intentos,          │
│              mantener en cola               │
│         SÍ → marcar como 'sincronizado'     │
│              en SQLite local                │
└─────────────────────────────────────────────┘
```

### 5.2 Manejo de conflictos

Si dos cajas registran una venta del mismo producto al mismo tiempo, el servidor usa el **timestamp** de la caja (no de llegada al servidor) para ordenar las transacciones. El campo `UNIQUE(caja_id, venta_local_id)` en PostgreSQL garantiza que no se dupliquen registros aunque la caja envíe el mismo lote dos veces.

---

## 6. Alcance del Prototipo Académico

Para la entrega académica se construye **1 caja completa** que demuestra el ciclo:

```
Registrar venta (offline) → Cola de sync → Conexión restaurada → 
Sync automático → Verificación en PostgreSQL
```

Las 14 cajas escalan el mismo patrón: instalar la misma app Electron en cada PC y configurar el ID de caja. No requiere desarrollo adicional.

---

## 7. Walkthrough SQA2 — Revisión de Arquitectura

| Criterio | ¿Cumple? | Observación |
|----------|----------|-------------|
| La arquitectura resuelve el problema raíz identificado | ✅ | Elimina el punto único de fallo |
| El stack es justificado técnicamente | ✅ | Sección 3 justifica cada decisión |
| Los diagramas son consistentes entre sí | ✅ | Diagrama de arquitectura y esquemas BD son coherentes |
| El motor de sync maneja casos de error | ✅ | Sección 5.2 cubre conflictos y reintentos |
| El alcance académico es realista | ✅ | 1 caja demo es suficiente para demostrar el patrón |

**Revisores:** Emanuel Lazo (autor) · Dylan Chanco (revisor)  
**Resultado:** ✅ Aprobado

---

*Documento de Arquitectura v1.0 — Grupo 01 · UPLA · MDS 2026-I*

