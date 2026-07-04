# 🛒 Sistema POS Offline-First — Tienda Plasita

**Universidad Peruana Los Andes · Facultad de Ingeniería de Sistemas**  
**Metodología de Desarrollo de Software · V Semestre 2026-I**  
**Grupo 01**

---

## 👥 Integrantes

| Nombre | Rol en el proyecto |
|--------|-------------------|
| Casimiro Laura, Kimberly | Líder / Analista de Requisitos |
| Lazo Eulogio, Emanuel Alexander | Arquitecto / Desarrollador |
| Chanco Torres, Dylan Diego | Diseño UI / Frontend |
| Ccoñas Gomez, Kevin Yeison | Requisitos / Tester |

---

## 🏪 Contexto del Problema

**Cliente:** Tienda Plasita — Huancayo, Junín  
**Problema:** El sistema VipSystem actual opera con un servidor centralizado 
que colapsa cuando las 14 cajas trabajan simultáneamente. Si el servidor 
falla, ninguna caja puede registrar ventas.

**Impacto real:**
- Pérdida de ventas en horas punta
- Pagos Yape/Plin interrumpidos por conectividad del sótano
- Inventario registrado ≠ stock físico real

---

## 💡 Solución Propuesta

Arquitectura **offline-first distribuida**: cada caja opera de forma 
autónoma con su propia base de datos local. Registra ventas sin necesidad 
de conexión al servidor. Cuando la red se restaura, sincroniza 
automáticamente con el servidor central de la administradora.
---

## 🔧 Stack Tecnológico

| Capa | Tecnología | Justificación |
|------|-----------|---------------|
| App de caja | Electron + React | Desktop en JavaScript, sin nuevo lenguaje |
| BD local caja | SQLite | Embebida, sin instalación, opera offline |
| Servidor sync | Node.js + Express | Mismo ecosistema JS del equipo |
| BD central | PostgreSQL | Open source, robusto, costo cero |
| Pagos digitales | Adapter Yape/Plin | Simulado en desarrollo, real en producción |

---

## 📋 Metodología: Modelo Incremental

Este proyecto aplica el **Modelo Incremental** porque:
- Los requisitos del cliente no están completamente definidos al inicio
- El sistema antiguo debe seguir operando en paralelo durante la transición
- Cada incremento entrega valor verificable al cliente
- Permite incorporar feedback real entre incrementos

### Incremento 1 — Semanas 1 a 6
> **Entregable:** ERS validada + Prototipo navegable (Figma)

- Diagnóstico organizacional completo
- Especificación de Requisitos de Software (ERS v1.0)
- Wireframes de pantallas principales validados con el cliente
- Decisión de arquitectura documentada

### Incremento 2 — Semanas 7 a 12
> **Entregable:** Módulo POS funcional (1 caja demo) + Informe final

- App Electron + React instalable
- SQLite local con ciclo de venta offline
- Motor de sincronización con servidor central
- Pruebas de carga con JMeter (14 usuarios simultáneos)

---

## 📁 Estructura del Repositorio
pos-plasita-mds2026/
-├── docs/                    # Documentación del proyecto
-│   ├── 01-diagnostico.md    # Diagnóstico organizacional
-│   ├── 02-ERS.md            # Especificación de Requisitos
-│   ├── 03-arquitectura.md   # Diseño de arquitectura
-│   ├── 04-plan-calidad.md   # ISO 25010 + SQA Shift Left
-│   └── 05-casos-de-prueba.md
-├── incremento-1/            # Artefactos del Incremento 1
-├── incremento-2/            # Artefactos del Incremento 2
-└── assets/diagramas/        # Imágenes y diagramas

---

## 📊 Seguimiento del Proyecto

El avance se gestiona mediante **GitHub Projects** con tablero Kanban.  
Cada tarea está asignada a un integrante y vinculada a su incremento.

- 🔵 Issues activas → tablero del proyecto
- 📌 Milestones → Cierre Incremento 1 · Cierre Incremento 2
- 🏷️ Etiquetas → `incremento-1` `incremento-2` `sqa` `análisis` `diseño`

---

## 📐 Plan de Calidad — ISO/IEC 25010

Características críticas del sistema:

| Característica | Nivel | Métrica |
|---------------|-------|---------|
| Confiabilidad | 5/5 ★ | Disponibilidad ≥ 99.5% · Recuperación ≤ 5 min |
| Eficiencia de Desempeño | 5/5 ★ | 14 usuarios simultáneos · Respuesta ≤ 2 seg |
| Seguridad | 5/5 ★ | 0 vulnerabilidades críticas OWASP Top 10 |

---

*Proyecto académico — Universidad Peruana Los Andes · 2026-I*
