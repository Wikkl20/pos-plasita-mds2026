# 01 · Diagnóstico Organizacional — Tienda Plasita

**Proyecto:** Sistema POS Offline-First  
**Grupo:** 01 · Metodología de Desarrollo de Software · V Semestre 2026-I  
**Versión:** 1.0 · Mayo 2026

---

## 1. Contexto de la Organización

| Atributo | Detalle |
|----------|---------|
| **Nombre** | Plasita |
| **Ubicación** | Huancayo, Región Junín, Perú |
| **Rubro** | Comercio minorista — artículos de primera necesidad y consumo masivo |
| **Antigüedad** | 7 años de operación |
| **Tamaño** | 60 empleados |
| **Infraestructura** | Local de 1 500 m², área de cajas en sótano |
| **Soporte TI** | 1 ingeniero de sistemas interno |

---

## 2. Sistema Actual — VipSystem

Tienda Plasita opera actualmente con el sistema **VipSystem**, solución POS instalada hace 5 años. Su arquitectura es **cliente-servidor centralizado**: todas las cajas se conectan en tiempo real a un único servidor central ubicado en la sala de administración.

### Descripción técnica del sistema actual

```
[Caja 1] ──┐
[Caja 2] ──┤
[Caja 3] ──┤──→ [SERVIDOR CENTRAL VipSystem] ──→ [Base de datos única]
   ...     │         (punto único de fallo)
[Caja 14]──┘
```

Cada transacción de cada caja requiere comunicación constante con el servidor. Si el servidor no responde, la caja no puede procesar ninguna operación.

---

## 3. Diagnóstico: Problemas Identificados

### 3.1 Problema Principal — Punto único de fallo arquitectónico

> **Este es el problema raíz. No es un problema de software obsoleto, es un problema de arquitectura.**

El servidor centralizado colapsa cuando las 14 cajas operan simultáneamente en horas punta (mediodía y tarde). Al colapsar el servidor, **ninguna caja puede registrar ventas**, generando colas, pérdida de clientes y pérdida directa de ingresos.

**Frecuencia:** 3 a 4 veces por semana en horas punta.  
**Impacto estimado:** 20 a 45 minutos de inactividad por evento.

### 3.2 Problemas Derivados

| ID | Problema | Impacto |
|----|----------|---------|
| P1 | Servidor colapsa con 14 cajas simultáneas | Todas las cajas dejan de operar |
| P2 | Conectividad inestable en el sótano | Pagos Yape/Plin se interrumpen |
| P3 | Inventario registrado ≠ stock físico real | Decisiones de compra incorrectas |
| P4 | Sin modo offline en ninguna caja | 0% de resiliencia ante fallas |
| P5 | Sin reportes en tiempo real para administración | Gestión reactiva, no preventiva |

### 3.3 Análisis de Causa Raíz

Aplicando el modelo de contexto (Sommerville, 2016):

- **Causa técnica:** Arquitectura monolítica centralizada sin redundancia ni tolerancia a fallos.
- **Causa operativa:** Alta concurrencia de 14 cajas sin que el hardware del servidor pueda sostenerla.
- **Causa de conectividad:** El sótano donde operan las cajas tiene señal WiFi débil e inestable.

---

## 4. Requisitos de Alto Nivel

A partir del diagnóstico, se identifican los siguientes requisitos de alto nivel que debe cumplir el nuevo sistema:

| ID | Requisito | Prioridad |
|----|-----------|-----------|
| RAN-01 | Las cajas deben operar sin depender del servidor central | Alta |
| RAN-02 | Las ventas deben registrarse localmente y sincronizarse cuando haya red | Alta |
| RAN-03 | El sistema debe soportar 14 usuarios simultáneos sin degradación | Alta |
| RAN-04 | Integración con pagos digitales Yape y Plin | Alta |
| RAN-05 | Inventario consolidado visible en tiempo real para administración | Media |
| RAN-06 | Reportes de ventas por caja, turno y día | Media |
| RAN-07 | Migración progresiva sin detener operaciones de la tienda | Alta |

---

## 5. Justificación del Nuevo Sistema

El nuevo sistema **no reemplaza VipSystem con otra solución centralizada**. Resuelve el problema arquitectónico desde la raíz: cada caja pasa a ser un nodo autónomo que opera de forma independiente, sincronizando con el servidor central únicamente cuando la conexión está disponible.

Este patrón se denomina **arquitectura offline-first** y es el estándar utilizado por sistemas POS modernos en entornos con conectividad intermitente.

---

## 6. Alcance del Proyecto

### Dentro del alcance
- Módulo de punto de venta (registro de ventas, productos, cantidades, totales)
- Base de datos local por caja (SQLite)
- Motor de sincronización con servidor central
- Módulo de pagos digitales (Yape/Plin — simulado en desarrollo)
- Dashboard de administración (ventas consolidadas, inventario)

### Fuera del alcance
- Módulo de recursos humanos (planillas, asistencia)
- Módulo de créditos o préstamos a clientes
- Integración con proveedores o sistema de compras
- Aplicación móvil para clientes finales

---

*Documento elaborado por Grupo 01 — UPLA · MDS 2026-I*

