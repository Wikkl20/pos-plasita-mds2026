# 04 · Plan de Calidad — ISO/IEC 25010 + SQA Shift Left

**Proyecto:** Sistema POS Offline-First — Tienda Plasita  
**Versión:** 1.0 · Mayo 2026

---

## 1. Perfil de Calidad ISO/IEC 25010

Las 8 características de calidad del estándar se evalúan y priorizan según el contexto de Plasita.

### 1.1 Características Críticas (nivel 5/5)

Estas tres características son **no negociables**. Si el sistema falla en alguna de ellas, el problema de Plasita no queda resuelto.

---

#### ★ Confiabilidad — 5/5

**¿Por qué es crítica?**  
El problema principal de Plasita es que el sistema actual falla y detiene todas las cajas. El nuevo sistema debe ser el opuesto: que nunca deje de funcionar aunque haya fallas externas.

| Métrica | Valor objetivo | Herramienta de verificación |
|---------|---------------|----------------------------|
| Disponibilidad módulo de caja | ≥ 99.5% | Monitoreo de uptime (UptimeRobot) |
| Tiempo de recuperación ante falla del servidor | ≤ 5 minutos | Prueba de corte de red manual |
| Ventas perdidas por falla de red | 0 | Revisión de logs de cola_sync |

**Estrategia:** La confiabilidad se logra por diseño (arquitectura offline-first), no por corrección posterior.

---

#### ★ Eficiencia de Desempeño — 5/5

**¿Por qué es crítica?**  
14 cajas operando simultáneamente en horas punta es exactamente la condición que colapsa el sistema actual. El nuevo sistema debe soportarla sin degradación.

| Métrica | Valor objetivo | Herramienta de verificación |
|---------|---------------|----------------------------|
| Tiempo de respuesta por transacción | ≤ 2 segundos | Apache JMeter |
| Usuarios simultáneos sin degradación | 14 | Apache JMeter (test de carga) |
| La sincronización bloquea la UI | Nunca | Prueba manual + revisión de código |

**Estrategia:** La sincronización corre en un proceso en background (Worker Thread en Node.js). El cajero nunca espera a que se sincronice para registrar la siguiente venta.

---

#### ★ Seguridad — 5/5

**¿Por qué es crítica?**  
El sistema maneja datos de ventas y pagos digitales (Yape/Plin). Una vulnerabilidad puede exponer datos de clientes o permitir manipulación de registros.

| Métrica | Valor objetivo | Herramienta de verificación |
|---------|---------------|----------------------------|
| Vulnerabilidades críticas OWASP Top 10 | 0 | SonarCloud (automático en cada push) |
| Comunicación caja-servidor cifrada | HTTPS obligatorio | Revisión de configuración |
| Acceso sin autenticación | Imposible | Prueba de acceso sin credenciales |

**Estrategia:** SonarCloud se integra en el pipeline de GitHub Actions. Cada Pull Request es bloqueado automáticamente si introduce una vulnerabilidad crítica.

---

### 1.2 Características Importantes (nivel 4/5)

| Característica | Nivel | Métrica principal |
|---------------|-------|------------------|
| **Adecuación Funcional** | 4/5 | ≥ 95% de los RF verificados con casos de prueba |
| **Usabilidad** | 4/5 | Venta completada en ≤ 4 clics sin capacitación |
| **Mantenibilidad** | 4/5 | Cobertura de código ≥ 80% (Jest) |

### 1.3 Características Complementarias (nivel 3/5)

| Característica | Nivel | Justificación |
|---------------|-------|---------------|
| **Compatibilidad** | 3/5 | Solo se requiere Windows 10+ en las cajas |
| **Portabilidad** | 3/5 | Electron facilita instalación pero no se requiere multiplataforma |

---

## 2. Distribución de Esfuerzo de Calidad

Aplicando la **Regla 1-10-100** (Pressman, 2020):  
Detectar un defecto en requisitos cuesta 1 unidad. En desarrollo cuesta 10. En producción cuesta 100.

```
PREVENCIÓN (40%)          DETECCIÓN (45%)          CORRECCIÓN (15%)
─────────────────         ─────────────────         ─────────────────
SQA1: Revisión ERS        SQA3: Análisis estático   Corrección de bugs
SQA2: Walkthrough UML     SQA4: Code Review         Refactoring
Definición de DoD         SQA5: Pruebas aceptación
                          SQA6: Pruebas de carga
```

La mayor inversión está en **prevención y detección temprana** porque el costo de corregir en producción (con 60 empleados y 14 cajas operando) es inaceptable.

---

## 3. Plan SQA — Actividades Shift Left

"Shift Left" significa mover las actividades de calidad hacia el inicio del proceso, antes de que los defectos se encarezcan.

| ID | Actividad | Fase | ¿Shift Left? | Responsable | Herramienta |
|----|-----------|------|-------------|-------------|-------------|
| SQA1 | Revisión ERS con checklist IEEE 830 | Análisis | ✅ SÍ | Kevin + Kimberly | Checklist en doc 02-ERS.md |
| SQA2 | Walkthrough diagramas UML y MER | Diseño | ✅ SÍ | Dylan + Emanuel | Revisión presencial |
| SQA3 | Análisis estático en cada Pull Request | Construcción | ✅ SÍ | Todos | SonarCloud + GitHub Actions |
| SQA4 | Code Review obligatorio en cada PR | Construcción | ✅ SÍ | Todos (rotativo) | GitHub Pull Requests |
| SQA5 | Pruebas de aceptación con el cliente | Despliegue | ⚠️ PARCIAL | Kevin + Ing. interno | Casos de prueba (doc 05) |
| SQA6 | Pruebas de carga (14 usuarios) | Despliegue | ⚠️ PARCIAL | Emanuel | Apache JMeter |

> **SQA5 y SQA6 son parcialmente Shift Left** porque dependen de software funcional, pero se planifican desde el inicio (los casos de prueba están escritos antes del desarrollo).

---

## 4. Definition of Done (DoD)

Cada historia de usuario se considera **Terminada** (Done) únicamente cuando cumple **todos** los criterios siguientes:

```
✅ Código revisado mediante Pull Request (mínimo 1 revisor aprobó)
✅ SonarCloud no reporta issues nuevos de severidad Critical o Blocker
✅ Tests unitarios escritos con cobertura ≥ 80% del módulo
✅ Criterios de aceptación de la historia verificados manualmente
✅ Documentación actualizada si la historia modifica la API
✅ Issue cerrada en GitHub Projects y movida a columna "Terminado"
```

Sin DoD, "terminado" no tiene significado concreto. Este checklist es el contrato de calidad del equipo.

---

## 5. Flujo de Calidad en GitHub Actions

Cada vez que un integrante hace push a una rama o abre un Pull Request, se ejecuta automáticamente:

```
Push / Pull Request
        ↓
[GitHub Actions Pipeline]
   1. npm install
   2. npm test (Jest → cobertura ≥ 80%)
   3. SonarCloud Scan (OWASP Top 10)
        ↓
   ¿Pasó todo?
   SÍ → PR puede ser aprobado y mergeado
   NO → PR bloqueado hasta corregir
```

Este flujo implementa **SQA3 de forma automática** sin intervención manual.

---

*Plan de Calidad v1.0 — Grupo 01 · UPLA · MDS 2026-I*

