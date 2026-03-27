---
name: Template Splitter
version: 2.0.0
description: >
  Divide el HTML del template master en 3 partes exactas e inmutables:
  header_raw, body_zone y footer_raw.
  v2.0: Añade validación de truncación del template fuente y
  validación del banner CTA del footer.
---

trigger:
  - called_by: IF Email Generator Agent (step: split_template)

input:
  - raw_html: string  # HTML completo cargado desde Cloudinary

output:
  - header_raw: string   # HTML exacto del header (inmutable)
  - body_zone: string    # Zona editable del body (será reemplazada)
  - footer_raw: string   # HTML exacto del footer (inmutable)
---

# 🔪 ALGORITMO DE SPLIT

## Paso 1: Verificar integridad del template fuente ← NUEVO EN v2.0

Antes de hacer el split, verificar que el HTML recibido está completo:

```
VERIFICAR que raw_html.strip() termina con </html>
VERIFICAR que raw_html contiene <!-- 🔒 FOOTER_END --> o cierre equivalente
VERIFICAR que len(raw_html) > 30000 (tamaño mínimo esperado del master)
```

| Condición                                  | Acción                                      |
|--------------------------------------------|---------------------------------------------|
| No termina en `</html>`                    | ABORT — template truncado, reportar al agente|
| Tamaño < 30000 chars                       | WARNING — verificar si carga fue parcial    |
| Falta `<!-- 🔒 FOOTER_END -->`             | WARNING — continuar con fallback de markers |

> ❌ NO continuar el split si el template está truncado. Un template truncado
> producirá un footer incompleto que pasará al email final.

---

## Paso 2: Buscar markers en el HTML

### Markers primarios (preferidos):
```
HEADER: <!-- 🔒 HEADER_START --> ... <!-- 🔒 HEADER_END -->
BODY:   <!-- START BODY -->         ... <!-- END BODY -->
FOOTER: <!-- 🔒 FOOTER_START --> ... <!-- 🔒 FOOTER_END -->
```

### Markers secundarios (fallback):
```
HEADER: <!-- HEADER_START --> ... <!-- HEADER_END -->
BODY:   <!-- BODY_START -->   ... <!-- BODY_END -->
FOOTER: <!-- FOOTER_START --> ... <!-- FOOTER_END -->
```

---

## Paso 3: Extraer los 3 bloques

```
header_raw = todo el HTML desde el inicio del documento
             hasta (e incluyendo) <!-- 🔒 HEADER_END -->

body_zone  = todo el HTML entre
             <!-- START BODY --> y <!-- END BODY -->
             (incluyendo los comentarios markers)

footer_raw = todo el HTML desde <!-- 🔒 FOOTER_START -->
             hasta el final del documento (incluyendo </html>)
```

---

## Paso 4: Validación de split

```
1. Verificar que los 3 bloques no están vacíos
2. Verificar que header_raw contiene el logo de Interfondos
3. Verificar que footer_raw contiene:
   a. "Invierte en tus metas con Interfondos"     ← NUEVO v2.0
   b. "background-color:#0C3461" en el banner CTA ← NUEVO v2.0
   c. Texto legal (Interfondos S.A. SAF)
   d. Imagen de redes sociales (facebook.com)
   e. Sección de seguridad (recomendaciones)
   f. Cierre completo </html>                      ← NUEVO v2.0
4. Verificar que body_zone existe (puede estar vacía)
```

---

## Paso 5: Validación del banner CTA del footer ← NUEVO EN v2.0

Buscar en `footer_raw` el banner CTA e inspeccionar:

```python
# Pseudocódigo de validación
banner_text = extract_between(footer_raw, "FOOTER_START", "background-color")

if "Invierte en tus metas con Interfondos" not in footer_raw:
    LOG WARNING "Banner CTA con texto incorrecto — verificar template master"

if "#0C3461" not in banner_section:
    LOG WARNING "Banner CTA no usa color #0C3461 — puede mostrar color incorrecto"

if "#1EC170" in banner_section or "background-color: rgb(30, 193, 112)" in footer_raw:
    LOG ERROR "Banner CTA usa color antiguo #1EC170 — template master desactualizado"
    SUGGEST "Actualizar el template master en Cloudinary con el banner correcto"
```

---

## Paso 6: Output

```json
{
  "header_raw": "<!-- 🔒 HEADER_START --> ... <!-- 🔒 HEADER_END -->",
  "body_zone":  "<!-- START BODY --> ... <!-- END BODY -->",
  "footer_raw": "<!-- 🔒 FOOTER_START --> ... </html>"
}
```

---

# ═══════════════════════════════════════════════
# 🚫 REGLAS CRÍTICAS
# ═══════════════════════════════════════════════

❌ NO modificar ningún carácter del header_raw
❌ NO modificar ningún carácter del footer_raw
❌ NO interpretar ni "mejorar" el HTML encontrado
❌ NO agregar ni quitar espacios, saltos de línea ni indentación
❌ NO continuar si el template fuente está truncado (no termina en `</html>`)
✅ Retornar los strings EXACTAMENTE como están en el template fuente
✅ Reportar estado de banner CTA del footer al agente orquestador

---

# ═══════════════════════════════════════════════
# ⚠️ MANEJO DE ERRORES
# ═══════════════════════════════════════════════

| Situación                                   | Acción                                          |
|---------------------------------------------|-------------------------------------------------|
| Template truncado (no termina en `</html>`) | ABORT — reportar truncación, no continuar       |
| Marker HEADER no encontrado                 | ABORT — reportar marker faltante                |
| Marker FOOTER no encontrado                 | ABORT — reportar marker faltante                |
| Marker BODY no encontrado                   | Usar bloque central como body_zone              |
| HTML vacío o no cargó                       | ABORT — el fetch del template falló             |
| Header sin logo Interfondos                 | WARNING — continuar pero registrar alerta       |
| Footer sin "Invierte en tus metas..."       | WARNING — banner CTA desactualizado             |
| Footer con banner `#1EC170`                 | ERROR — template master desactualizado          |
| Footer sin `</html>` al final               | ERROR CRÍTICO — template fuente incompleto      |

---

# ═══════════════════════════════════════════════
# 📤 LOGGING
# ═══════════════════════════════════════════════

Reportar al agente:
- `template_chars`: longitud total del raw_html recibido
- `template_complete`: boolean (termina en `</html>`)
- `header_chars`: longitud en caracteres del header_raw
- `body_zone_found`: boolean
- `footer_chars`: longitud en caracteres del footer_raw
- `footer_has_correct_cta`: boolean ("Invierte en tus metas" presente)
- `footer_has_correct_color`: boolean (#0C3461 en banner)
- `markers_used`: primarios | secundarios | fallback
