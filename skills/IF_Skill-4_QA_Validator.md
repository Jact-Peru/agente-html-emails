---
name: Email QA Validator
version: 4.0.0
description: >
  Valida compatibilidad, estructura y calidad del HTML ensamblado.
  v4.0: Añade checks de truncación de archivo, banner CTA del footer,
  emojis Outlook-safe en cards, y apilado mobile de columnas múltiples.

trigger:
  - previous_step: assemble_email (Skill 0)

input:
  - html_email: string   # Email HTML completo ensamblado
  - header_raw: string   # Header original del template (para comparación)
  - footer_raw: string   # Footer original del template (para comparación)
---

# ═══════════════════════════════════════════════
# ✅ VALIDACIONES
# ═══════════════════════════════════════════════

## 🔒 GRUPO 0: Integridad del archivo (CRÍTICO) ← NUEVO EN v4.0

### 0.1 Archivo no truncado
Verificar que el HTML termina con `</html>` (ignorando espacios en blanco).

- ✅ OK: último token no-whitespace es `</html>`
- ❌ ERROR CRÍTICO: el archivo termina en texto o tag abierto → significa truncación
  - Causa común: el output del LLM fue cortado por límite de tokens
  - Fix: regenerar el email o completar manualmente el cierre

### 0.2 Marcador FOOTER_END presente
Verificar que existe `<!-- 🔒 FOOTER_END -->` antes del cierre `</html>`.

- ✅ OK: marker presente
- ❌ ERROR CRÍTICO: marker ausente → el footer está incompleto

### 0.3 Estructura de cierre válida
Verificar que la secuencia de cierre final es:
`</tbody> → </table> → </body> → </html>`

- ✅ OK: secuencia correcta
- ⚠️ WARNING: orden alterado

---

## 🔒 GRUPO 1: Integridad de Header y Footer (CRÍTICO)

### 1.1 Header intacto
Comparar el bloque entre `<!-- 🔒 HEADER_START -->` y `<!-- 🔒 HEADER_END -->`
del html_email contra el `header_raw` recibido.

- ✅ OK: son idénticos (char-by-char)
- ❌ ERROR CRÍTICO: cualquier diferencia → FAIL con detalle de dónde difieren

### 1.2 Footer intacto
Comparar el bloque entre `<!-- 🔒 FOOTER_START -->` y `<!-- 🔒 FOOTER_END -->`
del html_email contra el `footer_raw` recibido.
**Excepción:** si el `footer_raw` del master está truncado, validar contra la
estructura canónica documentada en IF_Skill-3_Master_Template v2.1.

- ✅ OK: son idénticos o cumplen estructura canónica
- ❌ ERROR CRÍTICO: footer incompleto o con contenido alterado

### 1.3 Sin duplicación de header
Verificar que el bloque del header aparece UNA SOLA VEZ en el html_email.

- ✅ OK: aparece exactamente 1 vez
- ❌ ERROR: aparece más de 1 vez

### 1.4 Sin contenido generado fuera del body
Verificar que todo el contenido generado se encuentra dentro de:
`<!-- START BODY -->` y `<!-- END BODY -->`

- ✅ OK
- ❌ ERROR: se detectó contenido inyectado fuera de los markers

---

## 🎯 GRUPO 1B: Banner CTA del Footer (CRÍTICO) ← NUEVO EN v4.0

### 1B.1 Texto correcto del banner
Verificar que el banner del footer contiene el texto exacto:
**"Invierte en tus metas con Interfondos"**

- ✅ OK: texto exacto presente
- ❌ ERROR CRÍTICO: texto diferente (ej. "Avanza hacia tus objetivos financieros")

### 1B.2 Color de fondo correcto del banner
Verificar que el banner CTA del footer usa `background-color:#0C3461`.

- ✅ OK: `#0C3461` encontrado en el banner
- ❌ ERROR CRÍTICO: banner con `#1EC170`, `#2DCE8A` u otro color

### 1B.3 Link del banner presente
Verificar que el banner contiene `href` válido (no vacío, no `#`).

- ✅ OK: href con URL válida
- ⚠️ WARNING: href="#" (placeholder)
- ❌ ERROR: href vacío o variable sin resolver

---

## 📐 GRUPO 2: Layout y Compatibilidad Outlook (CRÍTICO)

### 2.0 Wrapper de 600px en el body
Verificar que el contenido del body está envuelto en tabla centrada de 600px:
`<table align="center" width="600" style="width:600px; max-width:600px;" ...>`

- ✅ OK: wrapper con `width="600"` y `align="center"` encontrado
- ❌ ERROR CRÍTICO: body sin wrapper de 600px

### 2.1 Sin padding lateral estructural
Buscar en el BODY patrones de padding lateral en `<td>` de layout:

```
padding-left: [^0]   (padding-left distinto de 0 en td de layout)
padding-right: [^0]  (padding-right distinto de 0 en td de layout)
```

Excepción: padding lateral DENTRO de elementos inline (botones, badges).

- ✅ OK: márgenes laterales con columnas spacer `width="30"`
- ⚠️ WARNING: padding lateral < 15px en td de layout
- ❌ ERROR: padding lateral ≥ 15px en td de layout sin spacer columns

### 2.2 Columnas spacer presentes
Verificar que el body usa `<td width="30" ...>&nbsp;</td>` en bloques principales.

- ✅ OK: patrón spacer detectado
- ⚠️ WARNING: spacer ausente en algunos bloques
- ❌ ERROR: ningún spacer encontrado

### 2.3 Ancho máximo 600px

- ✅ OK: `max-width:600px` o `width="600"` en wrapper exterior
- ❌ ERROR: ancho mayor a 600px

### 2.4 Solo tablas para layout (no divs)

- ✅ OK: layout 100% en tablas
- ⚠️ WARNING: `<div>` detectado en zona de layout
- ❌ ERROR: estructura principal en divs

### 2.5 CSS inline únicamente

- ✅ OK: sin `<style>` en el body generado
- ❌ ERROR: `<style>` insertado en el body generado

### 2.6 Conditional comments para Outlook

- ✅ OK: `<!--[if mso]-->` en botones CTA; `<!--[if (gte mso 9)|(lte ie 8)]-->` en multi-col
- ⚠️ WARNING: falta en algunos bloques multi-columna
- ❌ ERROR: botón CTA sin VML fallback

---

## 📱 GRUPO 2B: Mobile Responsivo ← NUEVO EN v4.0

### 2B.1 Clase `card-stat` en columnas flotantes
Verificar que TODA tabla con `align="left"` y `width` menor a `50%` en el body
tiene la clase `card-stat`.

Detectar patrón: `align="left"` + `width="3` (inicio de 30%, 31%, 32%, etc.)
sin `class=".*card-stat.*"` → ERROR.

- ✅ OK: todas las columnas flotantes tienen `class="card-stat"` (o `class="mobile-full card-stat"`)
- ❌ ERROR CRÍTICO: columna flotante sin `card-stat` → no apilará en mobile

### 2B.2 CSS de card-stat definido
Verificar que el `<style>` del `<head>` incluye la regla `.card-stat` con:
- `width:100%!important`
- `display:block!important`
- `float:none!important`
- `margin-left:0!important`

- ✅ OK: regla `.card-stat` presente y completa
- ❌ ERROR CRÍTICO: regla ausente → las cards no apilan en mobile

### 2B.3 Contenedor de cards con text-align:center
Verificar que el `<td>` padre que contiene las columnas flotantes tiene
`style="text-align:center;"` o `class` equivalente.

- ✅ OK: contenedor con `text-align:center`
- ⚠️ WARNING: contenedor sin centrado → contenido puede quedar a la izquierda al apilar

### 2B.4 margin-bottom en cards apilables
Verificar que cada tabla `card-stat` tiene `margin-bottom:8px` o superior.

- ✅ OK: separación presente
- ⚠️ WARNING: sin `margin-bottom` → cards quedan pegadas al apilar

---

## 😀 GRUPO 2C: Emojis Outlook-Safe ← NUEVO EN v4.0

### 2C.1 HTML entities en celdas de cards
Verificar que los emojis dentro de `<td>` de tablas de stats/cards
usan HTML entities (`&#XXXXX;`) y NO caracteres Unicode directos.

Patrón de detección de emoji Unicode directo en TD:
Buscar caracteres con codepoint > U+1F000 dentro de `<td>...</td>`.

- ✅ OK: emojis como `&#128200;`, `&#9889;`, `&#127873;` etc.
- ❌ ERROR: emoji Unicode directo como `📈`, `⚡`, `🎁` dentro de `<td>`
  - En Outlook Desktop: el emoji se renderiza como `□` (cuadro vacío)

### 2C.2 Font-family Arial en celdas de emoji
Verificar que las celdas que contienen entities de emoji usan
`font-family:Arial,Helvetica,sans-serif` (no Sora) para garantizar rendering.

- ✅ OK: `font-family:Arial` en td de emoji
- ⚠️ WARNING: `font-family:Sora` en td de emoji → puede fallar en algunos clientes

### 2C.3 Emojis presentes en badges de sección
Verificar que los badges de sección (📊 DATOS DE TU FONDO, 🍀 LO QUE TE DA, etc.)
usan entities en lugar de Unicode.

- ✅ OK: entities encontradas
- ⚠️ WARNING: Unicode directo en badge (funciona en webmail pero falla en Outlook desktop)

---

## 🎨 GRUPO 3: Branding

### 3.1 Colores de marca aplicados
Paleta permitida: `#0C3461`, `#2DCE8A`, `#1A7A5E`, `#FFFFFF`, `#F5F7FA`,
`#444444`, `#222222`, `#333333`, `#E8ECF0`, `#E8F5EE`, `#666666`

- ✅ OK
- ⚠️ WARNING: color no reconocido (puede ser variante válida)
- ❌ ERROR: colores fuera de marca (rojo, negro absoluto en fondo)

### 3.2 Tipografía
Fuente principal: `Sora` con fallback `Helvetica,Arial,sans-serif`.

- ✅ OK
- ⚠️ WARNING: fuente alternativa sin fallback correcto

---

## 🔗 GRUPO 4: Links y CTAs

### 4.1 URLs de CTA presentes

- ✅ OK: `href` con URL válida
- ⚠️ WARNING: `href="#"` (placeholder, aceptable en draft)
- ❌ ERROR: `href` vacío

### 4.2 Sin links rotos obvios

- ✅ OK
- ❌ ERROR: `href="undefined"`, `href="null"`, `href="{{url}}"` sin resolver

---

## ⚡ GRUPO 5: Performance

### 5.1 Imágenes con dimensiones definidas

- ✅ OK: `width` y/o `height` explícitos en `<img>`
- ⚠️ WARNING: imagen sin dimensiones

### 5.2 Alt text en imágenes

- ✅ OK: `alt` presente
- ⚠️ WARNING: `alt` vacío (aceptable en decorativas)

---

# ═══════════════════════════════════════════════
# 📊 SCORING
# ═══════════════════════════════════════════════

```
score = 100
  - ERROR CRÍTICO (Grupo 0, 1, 1B, 2.1-2.2, 2B.1-2B.2): -30 cada uno
  - ERROR normal:                                          -15 cada uno
  - WARNING:                                               -5 cada uno
```

Umbrales:
- `score >= 85`: ✅ APROBADO — listo para envío
- `score 60-84`: ⚠️ REVISAR — correcciones menores recomendadas
- `score < 60`:  ❌ RECHAZADO — requiere corrección antes de enviar

---

# ═══════════════════════════════════════════════
# 📤 OUTPUT ESPERADO
# ═══════════════════════════════════════════════

```json
{
  "status": "ok | warning | error",
  "score": 88,
  "groups": {
    "file_integrity": {
      "status": "ok",
      "checks": [
        { "id": "0.1", "name": "Archivo no truncado",      "status": "ok" },
        { "id": "0.2", "name": "FOOTER_END presente",      "status": "ok" },
        { "id": "0.3", "name": "Cierre HTML válido",       "status": "ok" }
      ]
    },
    "header_footer_integrity": {
      "status": "ok",
      "checks": [
        { "id": "1.1", "name": "Header intacto",           "status": "ok" },
        { "id": "1.2", "name": "Footer intacto",           "status": "ok" },
        { "id": "1.3", "name": "Sin duplicación",          "status": "ok" },
        { "id": "1.4", "name": "Contenido en body",        "status": "ok" }
      ]
    },
    "footer_cta_banner": {
      "status": "ok",
      "checks": [
        { "id": "1B.1", "name": "Texto CTA correcto",      "status": "ok" },
        { "id": "1B.2", "name": "Color #0C3461",           "status": "ok" },
        { "id": "1B.3", "name": "Link presente",           "status": "ok" }
      ]
    },
    "layout_outlook": {
      "status": "warning",
      "checks": [
        { "id": "2.0", "name": "Wrapper 600px",            "status": "ok" },
        { "id": "2.1", "name": "Sin padding lateral",      "status": "ok" },
        { "id": "2.2", "name": "Spacer columns",           "status": "ok" },
        { "id": "2.3", "name": "Max width 600px",          "status": "ok" },
        { "id": "2.4", "name": "Solo tablas",              "status": "ok" },
        { "id": "2.5", "name": "CSS inline",               "status": "ok" },
        { "id": "2.6", "name": "Conditional comments",     "status": "warning",
          "detail": "Falta conditional comment en bloque 3-col" }
      ]
    },
    "mobile_responsive": {
      "status": "ok",
      "checks": [
        { "id": "2B.1", "name": "Clase card-stat",         "status": "ok" },
        { "id": "2B.2", "name": "CSS card-stat definido",  "status": "ok" },
        { "id": "2B.3", "name": "Contenedor centrado",     "status": "ok" },
        { "id": "2B.4", "name": "margin-bottom en cards",  "status": "ok" }
      ]
    },
    "emoji_outlook": {
      "status": "ok",
      "checks": [
        { "id": "2C.1", "name": "HTML entities en cards",  "status": "ok" },
        { "id": "2C.2", "name": "Font Arial en emoji td",  "status": "ok" },
        { "id": "2C.3", "name": "Entities en badges",      "status": "ok" }
      ]
    },
    "branding":     { "status": "ok" },
    "links":        { "status": "ok" },
    "performance":  { "status": "ok" }
  },
  "issues": [
    {
      "group": "layout_outlook",
      "id": "2.6",
      "severity": "warning",
      "detail": "Bloque de stats 3-col sin conditional comment Outlook.",
      "fix": "Agregar <!--[if (gte mso 9)|(lte ie 8)]> wrapping en ese bloque."
    }
  ],
  "recommendation": "Email apto para envío. Agregar conditional comment en bloque 3-col."
}
```

---

# ═══════════════════════════════════════════════
# 🚫 RESTRICCIONES
# ═══════════════════════════════════════════════

❌ NO modificar el HTML — solo reportar errores
❌ NO auto-corregir — solo diagnosticar
✅ Proveer ubicación específica (línea o bloque) para cada issue
✅ Proveer sugerencia de fix concisa para cada issue
✅ Reportar truncación como ERROR CRÍTICO del Grupo 0 antes que cualquier otro check
