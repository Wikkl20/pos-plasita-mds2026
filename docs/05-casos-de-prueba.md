# 05 · Casos de Prueba — SQA5 y SQA6

**Proyecto:** Sistema POS Offline-First — Tienda Plasita  
**Versión:** 1.0 · Mayo 2026  
**Responsable:** Ccoñas Gomez, Kevin Yeison

---

## 1. Casos de Prueba de Aceptación (SQA5)

Estos casos son ejecutados por el ingeniero interno de Plasita junto con Kevin (tester del grupo). Verifican que el sistema cumple los requisitos funcionales desde la perspectiva del usuario real.

### CP-01 · Venta offline completa

| Campo | Detalle |
|-------|---------|
| **ID** | CP-01 |
| **Historia de usuario** | HU-01 |
| **Requisito** | RF-01 |
| **Precondición** | La caja está iniciada, cajero autenticado, servidor desconectado |

**Pasos:**
1. Desconectar el cable de red de la PC de caja
2. Abrir la app POS-Plasita
3. Verificar que el indicador de estado muestra 🔴 "Sin conexión"
4. Escanear el código de barras del producto "Arroz 1kg"
5. El sistema debe mostrar nombre, precio (S/ 3.50) y subtotal
6. Ingresar cantidad: 2
7. El sistema debe mostrar total: S/ 7.00
8. Seleccionar método de pago: Efectivo
9. Confirmar venta

**Resultado esperado:**
- ✅ La venta se registra correctamente en SQLite local
- ✅ Aparece en el resumen del turno con estado "Pendiente de sync"
- ✅ La app NO muestra error ni se congela
- ✅ El indicador sigue mostrando el conteo de ventas pendientes

**Resultado real:** _[completar al ejecutar]_  
**¿Pasó?** ⬜ SÍ / ⬜ NO  
**Observaciones:** _______________

---

### CP-02 · Sincronización automática al restaurar conexión

| Campo | Detalle |
|-------|---------|
| **ID** | CP-02 |
| **Historia de usuario** | HU-02 |
| **Requisito** | RF-02 |
| **Precondición** | CP-01 ejecutado exitosamente, 3 ventas pendientes en cola |

**Pasos:**
1. Reconectar el cable de red de la PC de caja
2. Esperar hasta 60 segundos
3. Verificar el indicador de estado en la app
4. Abrir el dashboard de administración en la PC administradora
5. Verificar las ventas en el panel

**Resultado esperado:**
- ✅ El indicador cambia a 🟢 "Sincronizado" sin acción del cajero
- ✅ El contador de ventas pendientes vuelve a 0
- ✅ Las 3 ventas aparecen en el dashboard de administración
- ✅ Los totales coinciden exactamente con lo registrado offline

**Resultado real:** _[completar al ejecutar]_  
**¿Pasó?** ⬜ SÍ / ⬜ NO  
**Observaciones:** _______________

---

### CP-03 · Pago con Yape (simulado)

| Campo | Detalle |
|-------|---------|
| **ID** | CP-03 |
| **Historia de usuario** | HU-03 |
| **Requisito** | RF-05 |
| **Precondición** | Caja conectada al servidor, cajero autenticado |

**Pasos:**
1. Iniciar una venta con producto "Leche Gloria 1L" — S/ 4.20
2. Seleccionar método de pago: Yape
3. El sistema debe mostrar un código QR en pantalla
4. Esperar la simulación de confirmación (el adapter confirma en 3 segundos)
5. Verificar que la venta se registra como completada

**Resultado esperado:**
- ✅ QR aparece en ≤ 2 segundos
- ✅ Confirmación simulada se recibe en ≤ 5 segundos
- ✅ Venta registrada con método_pago = 'yape'
- ✅ Monto correcto en el resumen del turno

**Resultado real:** _[completar al ejecutar]_  
**¿Pasó?** ⬜ SÍ / ⬜ NO  
**Observaciones:** _______________

---

### CP-04 · Cierre de turno

| Campo | Detalle |
|-------|---------|
| **ID** | CP-04 |
| **Historia de usuario** | — |
| **Requisito** | RF-04 |
| **Precondición** | Al menos 5 ventas registradas en el turno |

**Pasos:**
1. Hacer clic en "Cerrar turno"
2. Revisar el resumen generado

**Resultado esperado:**
- ✅ Muestra total de ventas del turno
- ✅ Desglose por método de pago (efectivo / Yape / Plin)
- ✅ Número de transacciones correcto
- ✅ Indica si hay ventas pendientes de sync

**Resultado real:** _[completar al ejecutar]_  
**¿Pasó?** ⬜ SÍ / ⬜ NO  
**Observaciones:** _______________

---

### CP-05 · Intento de acceso sin autenticación

| Campo | Detalle |
|-------|---------|
| **ID** | CP-05 |
| **Requisito** | RNF-03 (Seguridad) |
| **Precondición** | App instalada, sin sesión iniciada |

**Pasos:**
1. Abrir la app POS-Plasita
2. Intentar acceder a la pantalla de ventas sin ingresar credenciales
3. Intentar acceder directamente a la URL del dashboard sin autenticación

**Resultado esperado:**
- ✅ La app no permite acceso sin credenciales válidas
- ✅ El dashboard redirige al login si no hay sesión activa
- ✅ No se muestra ningún dato sensible antes de autenticarse

**Resultado real:** _[completar al ejecutar]_  
**¿Pasó?** ⬜ SÍ / ⬜ NO  
**Observaciones:** _______________

---

## 2. Plan de Pruebas de Carga (SQA6)

### Objetivo
Verificar que el sistema soporta 14 usuarios simultáneos (una por caja) sin que el tiempo de respuesta supere los 2 segundos, tal como especifica RNF-02.

### Herramienta
**Apache JMeter** — versión 5.6.x (gratuita, open source)

### Configuración del test

```
Thread Group:
  - Número de hilos (usuarios): 14
  - Ramp-up period: 10 segundos
    (1 usuario nuevo cada 0.7 seg, simula apertura gradual de cajas)
  - Loop count: 10
    (cada "usuario" realiza 10 transacciones)
  - Total transacciones: 140

Endpoints a probar:
  POST /sync          → envío de lote de ventas
  GET  /ventas        → consulta de ventas consolidadas
  GET  /productos     → catálogo de productos
```

### Métricas a medir

| Métrica | Umbral aceptable | Resultado esperado |
|---------|-----------------|-------------------|
| Tiempo de respuesta promedio | ≤ 2,000 ms | < 800 ms |
| Tiempo de respuesta P95 | ≤ 3,000 ms | < 1,500 ms |
| Tasa de errores | 0% | 0% |
| Throughput | ≥ 10 req/seg | > 20 req/seg |

### Escenarios de prueba

**Escenario 1 — Carga normal:** 14 usuarios enviando 1 venta cada 30 segundos (ritmo de hora normal).

**Escenario 2 — Hora punta:** 14 usuarios enviando 5 ventas por minuto simultáneamente (ritmo de mediodía en Plasita).

**Escenario 3 — Sync masivo:** Las 14 cajas reconectan al mismo tiempo después de 1 hora offline y envían 50 ventas acumuladas cada una (700 transacciones simultáneas). Este es el escenario más exigente y el más relevante para el problema de Plasita.

### Reporte esperado
JMeter genera automáticamente un reporte HTML con gráficos de tiempo de respuesta, throughput y distribución de errores. Este reporte se guarda en `/incremento-2/resultados-jmeter/` del repositorio.

---

## 3. Resumen de Cobertura

| Requisito | CP que lo verifica | Estado |
|-----------|-------------------|--------|
| RF-01 (venta offline) | CP-01 | ⬜ Pendiente |
| RF-02 (sync automático) | CP-02 | ⬜ Pendiente |
| RF-04 (cierre turno) | CP-04 | ⬜ Pendiente |
| RF-05 (pago Yape) | CP-03 | ⬜ Pendiente |
| RNF-02 (14 usuarios) | SQA6 JMeter | ⬜ Pendiente |
| RNF-03 (seguridad) | CP-05 | ⬜ Pendiente |

---

*Casos de Prueba v1.0 — Grupo 01 · UPLA · MDS 2026-I*

