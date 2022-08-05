---
layout: post
title: Multiples archivos en javascript
tags: [javascript]
---

Leer un conjunto de archivos en formato json, cada uno de estos archivos contienen un parámetro con el nombre de una imagen, en este caso un png. La idea principal es que al terminar de cargar todos los archivos invoque una función predefinida (callback).

## Datos iniciales 

```json
{
  "frames": {
    "Grass1 0.png": {
      "frame": { "x": 0, "y": 0, "w": 512, "h": 512 },
        "rotated": false,
        "trimmed": false,
        "spriteSourceSize": { "x": 0, "y": 0, "w": 512, "h": 512 },
        "sourceSize": { "w": 512, "h": 512 },
        "duration": 100
    }
  },
    "meta": {
      "app": "https://www.aseprite.org/",
      "version": "1.2.34.1-x64",
      "image": "Grass1-sheet.png",
      "format": "RGBA8888",
      "size": { "w": 3584, "h": 3072 },
      "scale": "1",
      "frameTags": [],
      "layers": [{ "name": "Layer", "opacity": 255, "blendMode": "normal" }],
      "slices": []
    }
}
```

## Solución 


## Ejemplo
