---
name: Brand Consistency Validator
description: Detecta inconsistencias entre diferentes fuentes de brand y su aplicación en el HTML generado.
---
trigger:
  - after: Brand Enforcer
  - before: IF Master Template
  - optional_after: HTML generation

input:
  - brand_asset_md
  - brand_embedded_md_optional
  - json_enriched_optional
  - html_output_optional

process:
  steps:

    - parse_brand_asset:
        action: convertir_md_a_json
        output: brand_asset_struct

    - parse_brand_embedded:
        condition: exists
        action: convertir_md_a_json
        output: brand_embedded_struct

    - comparar_fuentes:
        condition: brand_embedded_md_optional exists
        checks:
          - colores
          - tipografia
          - botones
          - tono
        output: diferencias_fuentes

    - validar_json_enriched:
        condition: exists
        checks:
          - colores_aplicados_vs_brand
          - uso_correcto_tipografia
          - consistencia_cta
        output: inconsistencias_enrichment

    - validar_html_final:
        condition: exists
        checks:
          - colores_en_html_vs_brand
          - estilos_inline_correctos
          - botones_consistentes
        output: inconsistencias_html

    - generar_score:
        formula:
          consistencia_total: >
            100 - (errores * 20 + warnings * 10)

rules:
  - no_modificar_datos: true
  - solo_analizar: true

output:
  format: json
  structure:
    status: ok | warning | error
    score: number
    issues:
      - type: source_conflict | style_mismatch | html_mismatch
        severity: warning | error
        detail: string
    recommendation: string
    
