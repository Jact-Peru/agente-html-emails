---
name: IF Master Template
version: 2.1.0
description: >
  Genera el bloque BODY del email usando el template map de Interfondos.
  Incluye reglas actualizadas para cards multi-columna con apilado mobile,
  emojis Outlook-safe, y banner CTA de footer correcto.
---
trigger:
  - called_by: IF Email Generator Agent (step: generate_body)
input:
  - json_enriched: json   # Contenido enriquecido con branding aplicado
  - body_zone: string     # Zona editable extraída del template master
output:
  - body_html: string     # Solo el bloque BODY (entre <!-- START BODY --> y <!-- END BODY -->)
---

# ═══════════════════════════════════════════════
# 🗺️ TEMPLATE MAP — BLOQUES DISPONIBLES
# ═══════════════════════════════════════════════

## Regla global de estructura
- Layout 100% tablas. NUNCA usar `<div>` como contenedor de layout.
- CSS inline únicamente en el BODY generado.
- Ancho máximo: 600px con wrapper centrado.
- Márgenes laterales: columnas spacer `width="30"` — NUNCA `padding-left/right`.
- Compatible Outlook: conditional comments `<!--[if mso]-->` en botones y columnas múltiples.

---

## 📐 WRAPPER OBLIGATORIO DEL BODY

Todo el contenido generado DEBE estar dentro de este wrapper:

```html
<!--[if (gte mso 9)|(lte ie 8)]><table width="600" align="center" cellpadding="0" cellspacing="0" border="0"><tr><td><![endif]-->
<table align="center" border="0" cellpadding="0" cellspacing="0" class="mobile-full"
       style="width:600px; max-width:600px; background-color:#FFFFFF;" width="600">
  <tbody>
    <!-- bloques aquí -->
  </tbody>
</table>
<!--[if (gte mso 9)|(lte ie 8)]></td></tr></table><![endif]-->
```

---

## 🧱 BLOQUE: SPACER

```html
<tr><td bgcolor="#FFFFFF" style="font-size:0; line-height:0; mso-line-height-rule:exactly; padding-top:24px;">&nbsp;</td></tr>
```

---

## 🧱 BLOQUE: PARAGRAPH / GREETING

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:12px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td style="font-family:Sora,Helvetica,Arial,sans-serif; font-size:15px; color:#444444; line-height:1.6;">
          Texto del párrafo aquí.
        </td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

---

## 🧱 BLOQUE: TITLE

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:12px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td style="font-family:Sora,Helvetica,Arial,sans-serif; font-size:24px; font-weight:700; color:#0C3461; line-height:1.3;">
          Título principal
        </td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

---

## 🧱 BLOQUE: SECTION BADGE

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:28px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td>
          <table role="presentation" cellspacing="0" cellpadding="0" border="0">
            <tr>
              <td bgcolor="#E8F5EE" style="border-radius:20px; padding:4px 14px;
                  font-family:Sora,Helvetica,Arial,sans-serif; font-size:11px;
                  font-weight:600; color:#0C3461; white-space:nowrap;">
                &#128202; ETIQUETA DE SECCIÓN
              </td>
            </tr>
          </table>
        </td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

> ⚠️ IMPORTANTE: Los emojis en badges deben ser HTML entities. Ver sección de emojis.

---

## 🧱 BLOQUE: ICON ITEM (checklist / beneficio)

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:12px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td>
          <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
            <tr>
              <td width="28" valign="top" style="padding-top:2px;">
                <table role="presentation" cellspacing="0" cellpadding="0" border="0"
                       width="24" height="24" style="background-color:#2DCE8A; border-radius:4px;">
                  <tr><td align="center" valign="middle"
                          style="font-family:Arial,sans-serif; font-size:13px;
                                 font-weight:700; color:#FFFFFF; line-height:24px;">&#10003;</td></tr>
                </table>
              </td>
              <td style="padding-left:10px; font-family:Sora,Helvetica,Arial,sans-serif;
                         font-size:14px; color:#444444; line-height:1.55; vertical-align:top;">
                <strong>Título del ítem</strong><br>Descripción del beneficio o punto.
              </td>
            </tr>
          </table>
        </td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

---

## 🧱 BLOQUE: 3-COL STATS ← ACTUALIZADO v2.1

### Reglas críticas para columnas múltiples:

1. Cada tabla de columna DEBE tener la clase `card-stat` para apilado mobile.
2. Los emojis SIEMPRE deben ser HTML entities (no Unicode directo).
3. El `td` contenedor de las 3 columnas debe tener `style="text-align:center;"`.
4. Agregar `margin-bottom:8px` a cada tabla para separación al apilar.

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:20px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td style="text-align:center;">
          <!--[if (gte mso 9)|(lte ie 8)]>
          <table width="540" cellpadding="0" cellspacing="0" border="0"><tr>
          <td width="173" valign="top"><![endif]-->

          <!-- CARD 1 — clase card-stat OBLIGATORIA -->
          <table role="presentation" cellspacing="0" cellpadding="0" border="0"
                 align="left" class="mobile-full card-stat" width="31.8%"
                 style="min-width:160px; background-color:#F5F7FA;
                        border-radius:6px; margin-bottom:8px;">
            <tr>
              <td align="center" style="padding:14px 8px 2px 8px;
                  font-family:Arial,Helvetica,sans-serif; font-size:24px; line-height:1.2;">
                &#128200;<!-- 📈 Entity: número/porcentaje -->
              </td>
            </tr>
            <tr>
              <td align="center" style="padding:4px 8px 4px 8px;
                  font-family:Sora,Helvetica,Arial,sans-serif; font-size:26px;
                  font-weight:700; color:#0C3461;">
                4.67%
              </td>
            </tr>
            <tr>
              <td align="center" style="padding:0 8px 14px 8px;
                  font-family:Sora,Helvetica,Arial,sans-serif; font-size:12px;
                  color:#666666; line-height:1.4;">
                Rentabilidad último año
              </td>
            </tr>
          </table>

          <!--[if (gte mso 9)|(lte ie 8)]></td><td width="7"></td>
          <td width="173" valign="top"><![endif]-->

          <!-- CARD 2 — clase card-stat OBLIGATORIA -->
          <table role="presentation" cellspacing="0" cellpadding="0" border="0"
                 align="left" class="mobile-full card-stat" width="31.8%"
                 style="min-width:160px; background-color:#F5F7FA;
                        border-radius:6px; margin-left:7px; margin-bottom:8px;">
            <tr>
              <td align="center" style="padding:14px 8px 2px 8px;
                  font-family:Arial,Helvetica,sans-serif; font-size:24px; line-height:1.2;">
                &#9889;<!-- ⚡ Entity: velocidad/acceso -->
              </td>
            </tr>
            <tr>
              <td align="center" style="padding:4px 8px 4px 8px;
                  font-family:Sora,Helvetica,Arial,sans-serif; font-size:18px;
                  font-weight:700; color:#2DCE8A;">
                Al día sig.
              </td>
            </tr>
            <tr>
              <td align="center" style="padding:0 8px 14px 8px;
                  font-family:Sora,Helvetica,Arial,sans-serif; font-size:12px;
                  color:#666666; line-height:1.4;">
                Cobras tu rescate al día hábil siguiente
              </td>
            </tr>
          </table>

          <!--[if (gte mso 9)|(lte ie 8)]></td><td width="7"></td>
          <td width="173" valign="top"><![endif]-->

          <!-- CARD 3 — clase card-stat OBLIGATORIA -->
          <table role="presentation" cellspacing="0" cellpadding="0" border="0"
                 align="left" class="mobile-full card-stat" width="31.8%"
                 style="min-width:160px; background-color:#F5F7FA;
                        border-radius:6px; margin-left:7px; margin-bottom:8px;">
            <tr>
              <td align="center" style="padding:14px 8px 2px 8px;
                  font-family:Arial,Helvetica,sans-serif; font-size:24px; line-height:1.2;">
                &#127873;<!-- 🎁 Entity: beneficio/regalo -->
              </td>
            </tr>
            <tr>
              <td align="center" style="padding:4px 8px 4px 8px;
                  font-family:Sora,Helvetica,Arial,sans-serif; font-size:26px;
                  font-weight:700; color:#0C3461;">
                0%
              </td>
            </tr>
            <tr>
              <td align="center" style="padding:0 8px 14px 8px;
                  font-family:Sora,Helvetica,Arial,sans-serif; font-size:12px;
                  color:#666666; line-height:1.4;">
                Comisión por rescate
              </td>
            </tr>
          </table>

          <!--[if (gte mso 9)|(lte ie 8)]></td></tr></table><![endif]-->
        </td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

---

## 🧱 BLOQUE: CTA BUTTON

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:24px; padding-bottom:8px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td align="center">
          <!--[if mso]>
          <v:roundrect xmlns:v="urn:schemas-microsoft-com:vml"
            href="https://erni.interfondos.com.pe"
            style="height:44px; v-text-anchor:middle; width:220px;"
            arcsize="10%" stroke="f" fillcolor="#2DCE8A">
            <w:anchorlock/>
            <center style="color:#FFFFFF; font-family:Sora,Helvetica,Arial,sans-serif;
                           font-size:15px; font-weight:700;">
              Texto del botón
            </center>
          </v:roundrect>
          <![endif]-->
          <!--[if !mso]><!-->
          <a href="https://erni.interfondos.com.pe" target="_blank"
             style="display:inline-block; background-color:#2DCE8A; color:#FFFFFF;
                    padding:12px 32px; text-decoration:none; border-radius:4px;
                    font-family:Sora,Helvetica,Arial,sans-serif; font-size:15px;
                    font-weight:700; mso-hide:all;">
            Texto del botón
          </a>
          <!--<![endif]-->
        </td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

---

## 🧱 BLOQUE: HIGHLIGHT BOX

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:20px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td bgcolor="#E8F5EE" style="border-radius:6px; padding:16px 0;">
          <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
            <tr>
              <td width="16" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
              <td align="center" style="font-family:Sora,Helvetica,Arial,sans-serif;
                  font-size:15px; font-weight:600; color:#0C3461; line-height:1.5;">
                Texto destacado principal.<br>
                <span style="color:#1A7A5E;">Texto secundario en verde.</span>
              </td>
              <td width="16" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
            </tr>
          </table>
        </td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

---

## 🧱 BLOQUE: DIVIDER

```html
<tr>
  <td bgcolor="#FFFFFF" style="padding-top:28px;">
    <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
      <tr>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td style="border-top:1px solid #E0E0E0; font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
        <td width="30" style="font-size:0; line-height:0; mso-line-height-rule:exactly;">&nbsp;</td>
      </tr>
    </table>
  </td>
</tr>
```

---

# ═══════════════════════════════════════════════
# 📱 REGLAS MOBILE (ACTUALIZADAS v2.1)
# ═══════════════════════════════════════════════

## CSS requerido en el <head> del template

El `@media only screen and (max-width:480px)` DEBE incluir estas reglas:

```css
/* CARD STACKING — CRÍTICO para columnas múltiples */
.card-stat {
  width: 100%!important;
  max-width: 100%!important;
  margin-left: 0!important;
  display: block!important;
  float: none!important;
  text-align: center!important;
}
.card-stat td {
  text-align: center!important;
}
```

### Reglas de apilado:
- Las tablas `align="left"` con `width < 50%` NO se apilan solas en mobile.
- SIEMPRE agregar `class="card-stat"` a cada tabla de columna flotante.
- El `<td>` contenedor de columnas flotantes DEBE tener `style="text-align:center;"`.
- Agregar `margin-bottom:8px` a cada card para separación al apilar.
- `margin-left:7px` en desktop se cancela a `0` en mobile via `.card-stat`.

---

# ═══════════════════════════════════════════════
# 😀 REGLAS DE EMOJIS (ACTUALIZADAS v2.1)
# ═══════════════════════════════════════════════

## Regla crítica: SIEMPRE usar HTML entities, NUNCA Unicode directo

Outlook (especialmente versiones de escritorio) ignora los emojis Unicode en celdas de tablas.
La solución es usar HTML numeric entities:

| Emoji | Entity     | Uso recomendado                        |
|-------|------------|----------------------------------------|
| 📈    | `&#128200;` | Rentabilidad, crecimiento, métricas   |
| ⚡    | `&#9889;`   | Velocidad, acceso rápido, rescate     |
| 🎁    | `&#127873;` | Beneficios, sin costo, regalo         |
| 📊    | `&#128202;` | Datos, estadísticas, sección de datos |
| 🍀    | `&#127808;` | Beneficios, ventajas del fondo        |
| 📰    | `&#128240;` | Noticias, contexto actual             |
| 🙋    | `&#128587;` | Decisiones del usuario                |
| 🛢️   | `&#128714;` | Petróleo, commodities                 |
| 🗳️   | `&#128379;` | Elecciones, votaciones                |
| 🏦    | `&#127974;` | Banco, tasas, institución financiera  |
| ✓     | `&#10003;`  | Check / confirmación                  |
| ✉️    | `&#9993;`   | Email / contacto                      |

### Regla de font para emojis en celdas:
```html
<!-- CORRECTO: font-family Arial primero para emoji rendering -->
<td style="font-family:Arial,Helvetica,sans-serif; font-size:24px; line-height:1.2;">
  &#128200;
</td>
```

---

# ═══════════════════════════════════════════════
# 🔒 REGLAS DE FOOTER (ACTUALIZADAS v2.1)
# ═══════════════════════════════════════════════

## El footer es INMUTABLE — NO re-generar

El footer se copia exactamente del `footer_raw` extraído por el Template Splitter.
Sin embargo, el agente DEBE verificar que el footer contiene estos bloques en orden:

### Estructura correcta del footer:

```
1. [BANNER CTA] → Fondo #0C3461, texto blanco, link a ERNI
2. [REDES SOCIALES] → Facebook, Instagram, TikTok, LinkedIn, YouTube
3. [LEGAL] → SMV logo + texto legal Interfondos + links
4. [INTELIGO] → Imagen "Somos parte de Inteligo Group"
5. [SEGURIDAD] → Sección de recomendaciones de seguridad (5 bullets)
6. [CIERRE] → </tbody></table></body></html>
```

## ⚠️ BANNER CTA DEL FOOTER — Especificación exacta:

```html
<!-- BANNER: fondo COMPLETAMENTE #0C3461 — sin botón anidado -->
<table align="center" border="0" cellpadding="0" cellspacing="0"
       class="mobile-full"
       style="width:600px; max-width:600px; background-color:#0C3461;"
       width="600">
  <tbody>
    <tr>
      <td align="center"
          style="padding:28px 20px; font-family:Sora,Helvetica,Arial,sans-serif;
                 font-size:17px; font-weight:700; color:#FFFFFF; line-height:1.4;">
        <a href="https://erni.interfondos.com.pe" target="_blank"
           style="color:#FFFFFF; text-decoration:none;
                  font-family:Sora,Helvetica,Arial,sans-serif;
                  font-size:17px; font-weight:700;">
          Invierte en tus metas con Interfondos
        </a>
      </td>
    </tr>
  </tbody>
</table>
```

### Reglas del banner:
- ❌ NO usar `#1EC170` ni `#2DCE8A` como fondo del banner de footer
- ❌ NO usar texto plano como "Avanza hacia tus objetivos financieros"
- ✅ Fondo SIEMPRE `#0C3461` (navy Interfondos)
- ✅ Texto SIEMPRE "Invierte en tus metas con Interfondos"
- ✅ Texto en blanco `#FFFFFF`, link a `https://erni.interfondos.com.pe`

## ⚠️ CIERRE DEL FOOTER — El archivo DEBE terminar con:

```html
        </tbody>
      </table>
      <!--[if (gte mso 9)|(lte ie 8)]></td></tr></table><![endif]-->
    </td>
  </tr>
</tbody>
</table>
<!-- 🔒 FOOTER_END -->
</tbody>
</table>
</body>
</html>
```

> ❌ ERROR CRÍTICO: Si el archivo termina en texto truncado o sin `</html>`, el email
> está incompleto. El QA Validator debe detectar esto como ERROR CRÍTICO.

---

# ═══════════════════════════════════════════════
# 🚫 RESTRICCIONES GLOBALES
# ═══════════════════════════════════════════════

❌ NO generar header ni footer — solo el body entre los markers
❌ NO usar `padding-left` / `padding-right` para márgenes laterales
❌ NO usar `<div>` como contenedor de layout
❌ NO usar emojis Unicode directos en celdas — usar HTML entities
❌ NO omitir `class="card-stat"` en tablas de columnas flotantes
❌ NO usar `#1EC170` como color del banner CTA del footer
✅ Usar SIEMPRE columnas spacer `width="30"` para márgenes
✅ Usar SIEMPRE `class="mobile-full card-stat"` en columnas múltiples
✅ Incluir SIEMPRE `<!--[if mso]-->` VML en botones CTA
