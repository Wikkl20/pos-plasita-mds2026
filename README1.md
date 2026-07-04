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
