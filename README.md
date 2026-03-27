# Auditoría de Accesibilidad — qdq Registro Anúnciate Gratis

Análisis de accesibilidad WCAG 2.1 AA del flujo de registro gratuito de qdq, realizado con Virtual Screen Reader mediante Claude Code.

**URL auditada:** https://www.qdq.com/registro/gratuito/step1

---

## Resultados

| | |
|--|--|
| **Violaciones** | 5 (2 críticas) |
| **Avisos** | 5 |
| **Correctos** | 6 |

### Violaciones

| # | Problema | WCAG | Impacto |
|---|----------|------|---------|
| 1 | Trampa de teclado en el selector de sector (1.540 ítems tabulables) | 2.1.1 · 2.1.2 · 2.4.3 | 🔴 Crítico |
| 2 | Errores de validación mostrados antes de interacción del usuario | 3.3.1 · 4.1.2 | 🟠 Grave |
| 3 | Mensajes de error no anunciados al lector de pantalla (sin live region) | 4.1.3 · 3.3.1 | 🟠 Grave |
| 4 | Orden de encabezados roto por el modal de cookies | 1.3.1 · 2.4.6 | 🟡 Moderado |
| 5 | Logo del header sin texto alternativo | 1.1.1 | 🟠 Grave |

---

## Contenido del repositorio

```
├── README.md                    # Este archivo
├── ACCESSIBILITY_AUDIT.md       # Informe completo con hallazgos y correcciones
└── accessibility-showcase.html  # Showcase visual interactivo (antes/después)
```

### `ACCESSIBILITY_AUDIT.md`
Informe detallado con:
- Descripción de cada hallazgo
- Código de ejemplo del problema actual
- Código corregido con la solución propuesta
- Tabla completa de los 16 hallazgos

### `accessibility-showcase.html`
Página HTML autocontenida con demostraciones visuales e interactivas de cada problema y su solución, incluyendo la salida simulada del lector de pantalla en cada caso.

---

## Metodología

- **Herramienta:** Virtual Screen Reader (MCP) sobre navegador headless
- **Estándar de referencia:** [WCAG 2.1 nivel AA](https://www.w3.org/TR/WCAG21/)
- **Patrones ARIA:** [APG — ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- **Navegación probada:** landmarks, encabezados, enlaces, controles de formulario, imágenes, navegación por teclado (Tab, Escape, flechas)

---

## Cómo ver el showcase

Abre `accessibility-showcase.html` directamente en el navegador:

```bash
open accessibility-showcase.html        # macOS
start accessibility-showcase.html       # Windows
xdg-open accessibility-showcase.html   # Linux
```

No requiere servidor ni dependencias.

---

*Auditoría realizada con [Claude Code](https://claude.ai/code) · Anthropic*
