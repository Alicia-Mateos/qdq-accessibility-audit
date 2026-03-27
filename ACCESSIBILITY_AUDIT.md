# Auditoría de Accesibilidad — QDQ Registro

**URL auditada:** `https://www.qdq.com/registro/gratuito/step1`
**Fecha:** 27 de marzo de 2026
**Estándar:** WCAG 2.1 nivel AA
**Herramienta:** Virtual Screen Reader (MCP)
**Auditor:** Claude Code (Anthropic)

---

## Resumen ejecutivo

| Métrica | Resultado |
|---------|-----------|
| Total de hallazgos | 15 |
| Violaciones | 4 |
| Avisos | 5 |
| Correctos | 6 |

La página presenta **dos bloqueadores críticos** que impiden completamente el uso del formulario de registro por parte de usuarios de teclado y lector de pantalla. El más grave es una trampa de teclado en el selector de sector que atrapa el foco en una lista de 1.540 elementos sin posibilidad de salida.

---

## Violaciones (prioridad de corrección)

### 🔴 F-007 & F-015 — Trampa de teclado en el selector de sector

**Impacto:** Crítico
**WCAG:** 2.1.1 Teclado · 2.1.2 Sin trampa de teclado · 2.4.3 Orden del foco

**Problema:**
El dropdown "Elige el sector que mejor describe tu empresa" expone sus 1.540 opciones como botones con `tabindex` positivo directamente en el DOM. Al pulsar `Tab`, el foco queda atrapado indefinidamente dentro de la lista y nunca alcanza los campos siguientes (`Página web`, botón `Ir a datos de empresa`). Pulsar `Escape` tampoco libera el foco.

Lo que anuncia el lector de pantalla:
```
button, Abanicos
button, Abastecedores de buques
button, Abastecimiento de agua
... [1.537 ítems más]
```

**Corrección:**
Implementar el patrón [ARIA Combobox (APG)](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/):

```html
<!-- ✗ Antes -->
<button tabindex="0">Abanicos</button>  <!-- ×1540 -->

<!-- ✓ Después -->
<div role="combobox" aria-expanded="true" aria-haspopup="listbox">
  <input aria-autocomplete="list" aria-activedescendant="opt1" />
  <ul role="listbox">
    <li role="option" tabindex="-1" id="opt1">Abanicos</li>
  </ul>
</div>
```

Comportamiento de teclado requerido:
- `↓ ↑` — navegan entre opciones
- `Enter` — selecciona la opción activa
- `Escape` — cierra el listbox y devuelve el foco al input
- `Tab` — cierra el listbox y mueve al siguiente campo del formulario

---

### 🔴 F-011 — Errores de validación mostrados en la carga inicial

**Impacto:** Grave
**WCAG:** 3.3.1 Identificación de errores · 4.1.2 Nombre, función, valor

**Problema:**
Los campos "Nombre de empresa" y "Sector" tienen `aria-invalid="true"` y mensajes de error visibles (`role="alert"`) desde el momento en que se carga la página, antes de cualquier interacción del usuario.

Lo que anuncia el lector de pantalla al cargar:
```
textbox, Nombre de la empresa, invalid, required
alert: Introduce el nombre de la empresa
textbox, Elige el sector, invalid, required
alert: Seleccione un sector
```

**Corrección:**
Activar la validación solo tras `blur` (campo vacío al salir) o al intentar `submit`.

```js
// ✗ Antes — validación al montar el componente
useEffect(() => { validateAll(); }, []);

// ✓ Después — validación tras interacción del usuario
<input
  onBlur={validateField}
  aria-invalid={hasError}
  aria-describedby={hasError ? 'error-id' : undefined}
/>
<div role="alert" id="error-id">
  {hasError ? 'Introduce el nombre de la empresa' : ''}
</div>
```

---

### 🟠 F-003 — Orden de encabezados roto por el modal de cookies

**Impacto:** Moderado
**WCAG:** 1.3.1 Información y relaciones · 2.4.6 Encabezados y etiquetas

**Problema:**
El primer encabezado encontrado en el DOM es un `<h2>` del modal de cookies ("Valoramos tu privacidad"), que aparece antes del `<h1>` de la página. Los usuarios que navegan por encabezados con la tecla `H` encuentran un h2 sin contexto jerárquico.

Orden actual en el DOM:
```
h2 "Valoramos tu privacidad"   ← modal de cookies (aparece primero)
h1 "Tu empresa en la Guía Útil"
h2 "Háblanos de tu empresa"
```

Orden esperado:
```
h1 "Tu empresa en la Guía Útil"
h2 "Háblanos de tu empresa"
h2 "Valoramos tu privacidad"   ← al final del DOM, como dialog
```

**Corrección:**
Mover el modal al final del `<body>` y añadir semántica de diálogo:

```html
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="cookie-title"
  aria-describedby="cookie-desc"
>
  <h2 id="cookie-title">Valoramos tu privacidad</h2>
  <p id="cookie-desc">…</p>
</div>
```

---

## Avisos (recomendados)

### F-001 — Título de página no verificable

**Impacto:** Moderado · **WCAG:** 2.4.2

El lector de pantalla solo anuncia "document" al cargar. Verificar que `<title>` contiene un texto descriptivo y único:

```html
<title>Registro gratuito — Paso 1: Datos de empresa | QDQ</title>
```

---

### F-005 — Enlaces con texto duplicado

**Impacto:** Moderado · **WCAG:** 2.4.4

Los enlaces "Política de cookies", "Política de privacidad" y "Aviso legal" aparecen dos veces (modal de cookies + footer). Si apuntan al mismo destino, es aceptable. Si apuntan a destinos distintos, diferenciarlos:

```html
<a href="/cookies" aria-label="Política de cookies - pie de página">
  Política de cookies
</a>
```

---

### F-009 — Formulario sin nombre accesible

**Impacto:** Moderado · **WCAG:** 1.3.1 · 4.1.2

El `<form>` se anuncia como "form" sin nombre. Añadir `aria-labelledby` apuntando al h2 existente:

```html
<h2 id="form-title">Háblanos de tu empresa</h2>
<form aria-labelledby="form-title">
```

---

### F-013 — Logo del header sin texto alternativo

**Impacto:** Grave · **WCAG:** 1.1.1

El banner está vacío en el árbol de accesibilidad; no se anuncia ningún logo ni enlace. Si el logotipo es el único identificador de la marca:

```html
<a href="/" aria-label="QDQ - Ir a inicio">
  <img src="logo.svg" alt="" />
</a>
```

---

### F-014 — Indicador de progreso sin nombres descriptivos

**Impacto:** Moderado · **WCAG:** 1.3.1 · 4.1.2

Los pasos anuncian "part 1", "part 2", "part 3". Implementar con semántica de lista y nombres reales:

```html
<nav aria-label="Progreso del registro">
  <ol>
    <li aria-current="step">Datos de empresa</li>
    <li>Datos de contacto</li>
    <li>Confirmación</li>
  </ol>
</nav>
```

---

## Elementos correctos

| ID | Elemento | WCAG |
|----|----------|------|
| F-002 | Landmarks completos: `banner`, `main`, `form`, `contentinfo`, `navigation` | 1.3.1, 2.4.1 |
| F-004 | H1 único y descriptivo: "Tu empresa en la Guía Útil" | 1.3.1, 2.4.6 |
| F-006 | Textos de enlace descriptivos (sin "clic aquí" ni "más info") | 2.4.4 |
| F-008 | Campo "Nombre de empresa": label visible, `required`, placeholder informativo | 1.3.1, 3.3.2, 4.1.2 |
| F-010 | Campo "Página web (opcional)": etiquetado correctamente, sin `required` | 1.3.1, 3.3.2, 4.1.2 |
| F-012 | Botón "Ir a datos de empresa": estado `disabled` anunciado correctamente | 4.1.2 |

---

## Tabla completa de hallazgos

| ID | Tipo | Impacto | WCAG | Elemento |
|----|------|---------|------|----------|
| F-001 | Aviso | Moderado | 2.4.2 | Título de página |
| F-002 | Correcto | — | 1.3.1, 2.4.1 | Landmark regions |
| F-003 | Violación | Moderado | 1.3.1, 2.4.6 | Orden de encabezados |
| F-004 | Correcto | — | 1.3.1, 2.4.6 | H1 único |
| F-005 | Aviso | Moderado | 2.4.4 | Enlaces duplicados |
| F-006 | Correcto | — | 2.4.4 | Texto de enlaces |
| F-007 | Violación | Crítico | 2.1.2, 4.1.2 | Trampa de teclado (dropdown) |
| F-008 | Correcto | — | 1.3.1, 3.3.2, 4.1.2 | Campo nombre de empresa |
| F-009 | Aviso | Moderado | 1.3.1, 4.1.2 | Formulario sin nombre |
| F-010 | Correcto | — | 1.3.1, 3.3.2, 4.1.2 | Campo página web |
| F-011 | Violación | Grave | 3.3.1, 4.1.2 | Errores de validación prematuros |
| F-012 | Correcto | — | 4.1.2 | Botón estado disabled |
| F-013 | Aviso | Grave | 1.1.1 | Logo sin alt text |
| F-014 | Aviso | Moderado | 1.3.1, 4.1.2 | Indicador de progreso |
| F-015 | Violación | Crítico | 2.1.1, 2.1.2, 2.4.3 | Trampa de teclado (navegación global) |

---

## Prioridad de corrección

```
BLOQUEADOR   F-007 / F-015  Trampa de teclado en selector de sector
GRAVE        F-011           Errores de validación en carga inicial
MODERADO     F-003           Orden de encabezados (modal cookies)
MODERADO     F-013           Logo sin alt text
MENOR        F-001           Título de página
MENOR        F-009           Formulario sin nombre accesible
MENOR        F-014           Indicador de progreso
MENOR        F-005           Enlaces duplicados
```

---

*Auditoría generada con Virtual Screen Reader MCP · Claude Code (Anthropic)*
