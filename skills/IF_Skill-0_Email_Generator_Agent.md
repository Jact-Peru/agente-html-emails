---
name: IF Email Generator Agent
version: 2.0.0
description: >
  Generador de emails HTML desde PDF/imagen usando template Interfondos.
  Preserva HEADER y FOOTER exactos del template fuente. Solo edita el BODY.
  Layout 100% tablas. Compatible con Outlook.
---

brand_control:
  source_of_truth: assets.brand.interfondos
  ignore_embedded_versions: true

# ─────────────────────────────────────────
# 🔐 PRIORIDADES DEL SISTEMA (CRÍTICO)
# ─────────────────────────────────────────
priority:
  1: template_integrity       # Header/Footer NUNCA se tocan
  2: outlook_compatibility    # Tablas, VML, conditional comments
  3: brand_guidelines         # Colores, tipografía, tono IF
  4: visual_fidelity          # Fidelidad al diseño fuente

# ─────────────────────────────────────────
# 📦 ASSETS EXTERNOS
# ─────────────────────────────────────────

assets:
  templates:
    if_master:
      id: if_master_template
      url: https://res.cloudinary.com/dgepa3uuk/raw/upload/v1774531818/HTML_Master_t8pzl7.html            
      type: html
      note: >
        Este archivo es la ÚNICA fuente de verdad para Header y Footer.
        Debe cargarse en su totalidad antes de cualquier generación.

  brand:
    interfondos:
      id: brand_interfondos
      url: https://res.cloudinary.com/dgepa3uuk/raw/upload/v1774383478/IF_Manual_de_marca_val3us.md
      type: markdown

# ─────────────────────────────────────────
# ⚙️ CONFIG GLOBAL
# ─────────────────────────────────────────
settings:
  max_width: 600
  layout: table_only          # NUNCA usar divs para layout
  css: inline_only
  outlook_mode: strict
  lateral_spacing: spacer_columns  # NUNCA padding-left/right para márgenes laterales
  spacer_width: 30            # px de columna spacer izq/der
  preserve_header: true
  preserve_footer: true
  editable_scope: body_only

# ─────────────────────────────────────────
# 🚀 FLUJO DE EJECUCIÓN
# ─────────────────────────────────────────
pipeline:

  # PASO 1: Cargar el template HTML COMPLETO desde Cloudinary
  - step: fetch_master_template
    action: http_fetch
    url: "{{assets.templates.if_master.url}}"
    output: raw_html_string
    on_error: ABORT  # Si no carga, detener todo el proceso
    note: >
      Guardar el HTML como string completo. NO interpretar ni transformar.

  # PASO 2: Cargar las brand guidelines
  - step: fetch_brand_guidelines
    action: http_fetch
    url: "{{assets.brand.interfondos.url}}"
    output: brand_guidelines_md
    on_error: ABORT

  # PASO 3: Dividir el template en 3 partes inmutables
  - step: split_template
    skill: Template Splitter
    input:
      raw_html: "{{raw_html_string}}"
    output:
      header_raw: string   # HTML exacto entre HEADER_START y HEADER_END
      body_zone: string    # Zona editable (entre START BODY y END BODY)
      footer_raw: string   # HTML exacto entre FOOTER_START y FOOTER_END
    rules:
      - header_raw es INMUTABLE: nunca modificar ni re-generar
      - footer_raw es INMUTABLE: nunca modificar ni re-generar
      - solo body_zone puede ser reemplazado

  # PASO 4: Extraer contenido del PDF/imagen fuente
  - step: extract_content
    skill: Extractor Visual
    input:
      file: "{{input.file}}"
    output:
      content_json: json_structure

  # PASO 5: Aplicar identidad de marca al contenido extraído
  - step: apply_brand
    skill: Brand Enforcer
    input:
      json_structure: "{{content_json}}"
      brand_guidelines_md: "{{brand_guidelines_md}}"
    output:
      json_enriched: json_enriched

  # PASO 6: Generar SOLO el bloque BODY usando Template Map
  - step: generate_body
    skill: IF Master Template
    input:
      json_enriched: "{{json_enriched}}"
      body_zone: "{{body_zone}}"
    output:
      body_html: string
    rules:
      - output es SOLO el contenido del body, no el email completo
      - usar exclusivamente columnas spacer para márgenes laterales
      - NO usar padding-left ni padding-right para espaciado estructural

  # PASO 7: Ensamblar el email final (COPY EXACTO de header + body generado + COPY EXACTO de footer)
  - step: assemble_email
    action: concatenate
    input:
      part_1: "{{header_raw}}"    # COPIA EXACTA del template fuente
      part_2: "{{body_html}}"     # Único contenido generado
      part_3: "{{footer_raw}}"    # COPIA EXACTA del template fuente
    output:
      html_output: string
    critical: >
      El email final = header_raw + body_html + footer_raw.
      Nunca re-generar ni parafrasear header_raw ni footer_raw.

  # PASO 8: Validar el output ensamblado
  - step: validate_output
    skill: Email QA Validator
    input:
      html_email: "{{html_output}}"
      header_raw: "{{header_raw}}"
      footer_raw: "{{footer_raw}}"
    output:
      qa_report: qa_results

# ─────────────────────────────────────────
# 📤 OUTPUT FINAL
# ─────────────────────────────────────────
output:
  html: "{{html_output}}"
  qa: "{{qa_results}}"

# ─────────────────────────────────────────
# 🚫 REGLAS GLOBALES (ENFORCED)
# ─────────────────────────────────────────
rules:
  - no_modificar_header: true
  - no_modificar_footer: true
  - no_regenerar_header: true       # NUEVA: no inventar header aunque el original sea difícil de leer
  - no_regenerar_footer: true       # NUEVA
  - solo_editar_body: true
  - usar_tablas: true
  - no_usar_padding_lateral: true   # NUEVA: spacer columns en lugar de padding-left/right
  - max_width_600: true
  - css_inline_only: true
  - outlook_conditional_required: true

# ─────────────────────────────────────────
# 🧠 COMPORTAMIENTO DEL AGENTE
# ─────────────────────────────────────────
behavior:
  on_fetch_failure:
    - si_template_no_carga: ABORT — no generar sin template fuente real
    - si_brand_no_carga: ABORT — no generar sin guías de marca

  on_content_extraction:
    - si_no_detecta_cta: no_inventar
    - si_falta_imagen: omitir_bloque
    - si_hero_en_fuente: convertir a title + paragraph (el hero pertenece al header)

  on_assembly:
    - SIEMPRE usar header_raw sin modificaciones
    - SIEMPRE usar footer_raw sin modificaciones
    - NUNCA mezclar HTML generado con partes del header o footer

  fallback:
    - si_split_falla: revisar markers y reportar al usuario — no continuar

# ─────────────────────────────────────────
# 🔍 LOGGING
# ─────────────────────────────────────────
logging:
  enabled: true
  level: detailed
  steps:
    - template_fetched           # URL + tamaño
    - template_split_ok          # Confirmación de los 3 bloques
    - header_chars: number       # Longitud del header extraído
    - footer_chars: number       # Longitud del footer extraído
    - content_extracted
    - brand_applied
    - body_generated
    - email_assembled
    - qa_validated
