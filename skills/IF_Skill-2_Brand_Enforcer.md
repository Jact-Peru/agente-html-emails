---
name: Brand Enforcer
description: Aplica identidad de marca al contenido usando el archivo Brand .md.
trigger:
  - previous_skill: Extractor Visual

input:
  - json_structure
  - brand_guidelines_md

process:
  steps:
    - aplicar_colores:
        source: brand_guidelines
    - aplicar_tipografia:
        jerarquia:
          - titulo
          - subtitulo
          - cuerpo
    - normalizar_cta:
        propiedades:
          - color_fondo
          - color_texto
          - padding
    - ajustar_tono:
        estilo: corporativo

rules:
  - no_cambiar_estructura: true
  - no_eliminar_contenido: true

output:
  format: json
  description: JSON enriquecido con estilos y reglas de marca

---

# 📄 SKILL 2 — Brand Enforcer

```md id="IF_Skill-2_Brand_Enforcer"
# Brand Enforcer

## 🎯 Propósito
Alinear el contenido extraído con la identidad de marca definida en el archivo IF_Manual_de_marca .md.

---

## 🧠 Contexto de uso
Se ejecuta después del extractor y antes del HTML.

---

## ⚙️ Instrucciones

1. Leer IF_Manual_de_marca .md
2. Aplicar:

### 🎨 Colores
- Reemplazar colores detectados por los oficiales

### 🔤 Tipografía
- Ajustar jerarquía:
  - Títulos
  - Subtítulos
  - Texto

### 🔘 Botones
- Normalizar:
  - Color fondo
  - Color texto
  - Padding

### 🧠 Tono
- Ajustar texto a:
  - Corporativo
  - Financiero
  - Formal

---

## 🚫 Restricciones

❌ No cambiar estructura del contenido  
❌ No eliminar información  

---

## 📤 Output esperado

JSON enriquecido con branding aplicado:

```json
{
  "sections": [
    {
      "type": "text",
      "style": "title-primary",
      "content": "..."
    }
  ]
}
