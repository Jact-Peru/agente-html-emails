---
name: Extractor Visual
description: Convierte un PDF o imagen en una estructura semántica organizada para email HTML.
trigger:
  - input_type: pdf
  - input_type: image (JPG, PNG)

input:
  - file
  - optional_text

process:
  steps:
    - detectar_bloques:
        items:
          - hero
          - titulos
          - parrafos
          - imagenes
          - botones
          - secciones

    - clasificar_contenido:
        formato: json
        estructura:
          hero: string
          sections:
            - type: text | image | cta
              title: string
              content: string
              src: string
              url: string

rules:
  - no_generar_html: true
  - no_aplicar_estilos: true
  - no_inventar_contenido: true

output:
  format: json
  description: Estructura limpia y jerárquica del contenido
---

# Extractor Visual (PDF / Imagen → Estructura)

## 🎯 Propósito
Analizar un PDF o imagen y convertirlo en una estructura semántica organizada para email HTML.

---

## 🧠 Contexto de uso
Se activa cuando el input es:
- PDF
- JPG / PNG
- Diseño visual

---
## 🔒 NORMALIZACIÓN DE HERO

Si se detecta un hero en input:

- NO generar bloque "hero"
- Convertir contenido en:
  - title
  - paragraph
  - cta (opcional)

El hero visual SIEMPRE pertenece al header (no dinámico)


## ⚙️ Instrucciones

1. Detectar bloques visuales:
   - Header (ignorar si ya existe en template)
   - Hero
   - Títulos
   - Párrafos
   - Botones
   - Imágenes
   - Secciones

2. Clasificar contenido:

```json
{
  "hero": "imagen_url",
  "sections": [
    {
      "type": "text",
      "title": "",
      "content": ""
    },
    {
      "type": "image",
      "src": ""
    },
    {
      "type": "cta",
      "text": "",
      "url": ""
    }
  ]
}
