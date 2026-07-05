# 02 · Especificación de Requisitos de Software (ERS v1.0)

**Proyecto:** Sistema POS Offline-First — Tienda Plasita  
**Estándar:** IEEE 830 adaptado  
**Versión:** 1.0 · Mayo 2026  
**Estado:** Validado con cliente ✅

---

## 1. Introducción

### 1.1 Propósito
Este documento especifica los requisitos funcionales y no funcionales del sistema POS Offline-First para Tienda Plasita. Está dirigido al equipo de desarrollo (Grupo 01) y al cliente (ingeniero interno de Plasita).

### 1.2 Ámbito del Sistema
El sistema se denomina **POS-Plasita** y cubre el registro de ventas en caja, la sincronización con el servidor central y el dashboard de administración. No cubre módulos de RRHH, créditos ni compras a proveedores.

### 1.3 Definiciones

| Término | Definición |
|---------|------------|
| **Caja** | PC de punto de venta donde opera el cajero |
| **Nodo autónomo** | Caja que opera sin depender del servidor central |
| **Cola de sync** | Lista de transacciones pendientes de sincronizar |
| **Offline-first** | Arquitectura donde la operación local es la principal |
| **Sync** | Proceso de envío de transacciones locales al servidor |

---

## 2. Descripción General

### 2.1 Perspectiva del Producto

POS-Plasita es un sistema nuevo que reemplaza a VipSystem. A diferencia de su predecesor, cada instancia de la aplicación (una por caja) opera de forma completamente autónoma. El servidor central recibe datos consolidados pero no es necesario para que las cajas funcionen.

### 2.2 Funciones Principales

```
POS-Plasita
├── Módulo Caja (Electron + React)
│   ├── Registro de ventas
│   ├── Gestión de productos
│   ├── Cierre de turno
│   └── Motor de sincronización
├── Módulo Servidor (Node.js + Express)
│   └── API de sincronización
└── Módulo Administración (Dashboard)
    ├── Ventas consolidadas
    └── Inventario en tiempo real
```

### 2.3 Usuarios del Sistema

| Rol | Descripción | Módulo que usa |
|-----|-------------|----------------|
| Cajero | Registra ventas en caja | App de caja |
| Administrador | Supervisa ventas e inventario | Dashboard |
| Ing. de sistemas | Gestiona el servidor y sincronización | Servidor + configuración |

---

## 3. Requisitos Funcionales

### RF-01 · Registro de Venta Offline
**Descripción:** El cajero puede registrar una venta completa (productos, cantidades, total, método de pago) sin que la caja tenga conexión al servidor central.  
**Precondición:** La aplicación está instalada y la base de datos local inicializada.  
**Flujo principal:**
1. El cajero selecciona productos o ingresa el código
2. El sistema calcula el total automáticamente
3. El cajero selecciona método de pago (efectivo / Yape / Plin)
4. El sistema guarda la venta en SQLite local y la añade a la cola de sync
5. El sistema muestra comprobante en pantalla

**Postcondición:** Venta guardada localmente, marcada como "pendiente de sync".

---

### RF-02 · Sincronización Automática
**Descripción:** Cuando la caja detecta conexión al servidor, envía automáticamente todas las ventas pendientes en la cola de sync.  
**Precondición:** Existen ventas en la cola con estado "pendiente".  
**Flujo principal:**
1. Cada 30 segundos el sistema verifica conectividad con el servidor
2. Si hay conexión, envía el lote de ventas pendientes vía `POST /sync`
3. El servidor las registra en PostgreSQL y responde con confirmación
4. El sistema marca las ventas como "sincronizadas" en SQLite local

**Manejo de errores:** Si el envío falla, la venta permanece en cola y se reintenta en el siguiente ciclo.

---

### RF-03 · Gestión de Productos
**Descripción:** El sistema mantiene un catálogo local de productos con código, nombre, precio y stock.  
**Requisitos específicos:**
- Búsqueda por código de barras o nombre
- Actualización de precios sincronizada desde el servidor
- Alerta visual cuando el stock local baja del mínimo configurado

---

### RF-04 · Cierre de Turno
**Descripción:** Al finalizar su turno, el cajero genera un resumen de las ventas realizadas.  
**Contenido del cierre:** Total de ventas, desglose por método de pago, número de transacciones, ventas pendientes de sync.

---

### RF-05 · Pago Digital (Yape / Plin)
**Descripción:** El sistema genera un código QR para pagos digitales y registra la confirmación.  
**Nota de implementación:** En el entorno de desarrollo se usa una capa adaptadora que simula la confirmación. En producción se reemplaza por la API real de cada proveedor sin modificar el módulo POS.

---

### RF-06 · Dashboard de Administración
**Descripción:** El administrador accede a un panel web que muestra ventas consolidadas de todas las cajas, inventario actual y alertas de stock crítico.

---

## 4. Requisitos No Funcionales

### RNF-01 · Confiabilidad ★ CRÍTICA
- Disponibilidad del módulo de caja: ≥ 99.5% (opera offline)
- Recuperación ante falla del servidor: ≤ 5 minutos
- Las ventas nunca se pierden, incluso si el servidor está caído

### RNF-02 · Eficiencia de Desempeño ★ CRÍTICA
- Tiempo de respuesta por transacción: ≤ 2 segundos
- Soporta 14 usuarios simultáneos sin degradación (verificado con JMeter)
- La sincronización no debe bloquear la interfaz del cajero

### RNF-03 · Seguridad ★ CRÍTICA
- Autenticación por usuario y contraseña en cada caja
- Comunicación cifrada entre caja y servidor (HTTPS)
- 0 vulnerabilidades críticas según OWASP Top 10 (verificado con SonarCloud)

### RNF-04 · Usabilidad
- El cajero debe poder completar una venta en ≤ 4 clics
- Interfaz en español, sin tecnicismos
- Indicador visual claro del estado de sincronización (🟢 sync / 🔴 offline)

### RNF-05 · Mantenibilidad
- Cobertura de código: ≥ 80% (verificado con Jest)
- Documentación de API actualizada en cada release
- Código revisado mediante Pull Requests en GitHub

---

## 5. Casos de Uso Principales

### CU-01 · Registrar Venta

```
Actor: Cajero
Precondición: Cajero autenticado en el sistema
Flujo:
  1. Cajero escanea o ingresa código de producto
  2. Sistema muestra nombre, precio y subtotal
  3. Cajero confirma cantidad
  4. Sistema actualiza total
  5. Cajero selecciona método de pago
  6. Sistema procesa y guarda la venta
  7. Sistema imprime/muestra comprobante
Postcondición: Venta en SQLite local + en cola de sync
Flujo alternativo: Si el producto no existe → sistema alerta al cajero
```

### CU-02 · Sincronizar Ventas

```
Actor: Sistema (automático)
Precondición: Hay ventas pendientes en la cola de sync
Flujo:
  1. Sistema detecta conexión disponible
  2. Sistema agrupa ventas pendientes en un lote
  3. Sistema envía lote al servidor vía API
  4. Servidor valida, procesa y confirma
  5. Sistema marca ventas como sincronizadas
Postcondición: Ventas registradas en PostgreSQL central
Flujo alternativo: Si falla → venta permanece en cola, reintento en 30 seg
```

### CU-03 · Cerrar Turno

```
Actor: Cajero / Administrador
Precondición: Turno activo con ventas registradas
Flujo:
  1. Usuario solicita cierre de turno
  2. Sistema genera resumen: total, métodos de pago, n° transacciones
  3. Sistema indica si hay ventas pendientes de sync
  4. Usuario confirma cierre
  5. Sistema guarda el acta de cierre
Postcondición: Turno cerrado, acta guardada localmente y sincronizada
```

---

## 6. Historias de Usuario

### HU-01 · Venta sin conexión
> Como **cajero de Plasita**, quiero **registrar ventas aunque el servidor esté caído**, para **no detener la atención al cliente en horas punta**.

**Criterios de aceptación:**
- [ ] La caja opera normalmente sin conexión al servidor
- [ ] La venta se guarda localmente y aparece en el resumen del turno
- [ ] Cuando se restaura la conexión, la venta se sincroniza sola

### HU-02 · Ver estado de sincronización
> Como **cajero**, quiero **ver en pantalla si mis ventas están sincronizadas**, para **saber si hay un problema de conexión antes de cerrar el turno**.

**Criterios de aceptación:**
- [ ] Indicador verde visible cuando todo está sincronizado
- [ ] Indicador rojo con contador de ventas pendientes cuando hay cola
- [ ] El indicador se actualiza en tiempo real sin recargar la app

### HU-03 · Pago con Yape
> Como **cajero**, quiero **generar un QR de Yape desde la pantalla de pago**, para **que el cliente pague sin necesitar efectivo**.

**Criterios de aceptación:**
- [ ] El QR se genera en ≤ 2 segundos
- [ ] El sistema espera confirmación antes de registrar la venta
- [ ] Si el pago no se confirma en 3 minutos, la transacción se cancela

### HU-04 · Reporte consolidado
> Como **administrador**, quiero **ver las ventas de todas las cajas en un solo dashboard**, para **tomar decisiones de inventario y personal en tiempo real**.

**Criterios de aceptación:**
- [ ] El dashboard muestra ventas del día agrupadas por caja
- [ ] Actualización automática cada 5 minutos
- [ ] Exportable a Excel

---

## 7. Matriz de Trazabilidad

| Requisito Alto Nivel | Req. Funcional | Req. No Funcional | Historia de Usuario |
|---------------------|---------------|-------------------|---------------------|
| RAN-01 (operar sin servidor) | RF-01 | RNF-01 | HU-01 |
| RAN-02 (sync automática) | RF-02 | RNF-02 | HU-02 |
| RAN-03 (14 usuarios) | RF-01, RF-02 | RNF-02 | — |
| RAN-04 (pagos digitales) | RF-05 | RNF-03 | HU-03 |
| RAN-05 (inventario consolidado) | RF-06 | RNF-02 | HU-04 |

---

## 8. Checklist SQA1 — Revisión IEEE 830

| Criterio | ¿Cumple? | Observación |
|----------|----------|-------------|
| Los requisitos son verificables | ✅ | Cada RF tiene criterios de aceptación medibles |
| Los requisitos no son ambiguos | ✅ | Cada término técnico está definido en Sección 1.3 |
| Los requisitos son completos | ✅ | Cubre funcional, no funcional, casos de uso e historias |
| Los requisitos son consistentes | ✅ | Sin contradicciones entre RF y RNF |
| Los requisitos están priorizados | ✅ | RNF críticos marcados con ★ |
| Trazabilidad a necesidades del cliente | ✅ | Matriz de trazabilidad en Sección 7 |
| Alcance definido claramente | ✅ | Sección 1.2 define dentro/fuera del alcance |

**Revisores:** Kevin Ccoñas (autor) · Kimberly Casimiro (revisora)  
**Fecha de revisión:** Mayo 2026  
**Resultado:** ✅ Aprobado sin observaciones críticas

---

*ERS v1.0 — Grupo 01 · UPLA · MDS 2026-I*

