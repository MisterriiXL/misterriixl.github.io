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
Originalmente pensé que cada vez que terminara de cargar un archivo, ya sea el json o el png hacer un callback, pero a medida que los archivos aumentaban el proceso se ponia mas complejo y engorroso, por lo tanto la unica opcion que encontre es hacer estas llamadas asíncronas a través de promise, estos fueron diseñados para saber si un proceso asíncrono finalizo con exito o con error.

Para la solución implemente 4 métodos:
 - Método para leer un archivo json.
 - Método para leer una imagen.
 - Método que relaciona el json con su imagen.
 - Método que lee la lista completa de archivos.

```javascript
readAll(callback) {
  const promises = this.files.map((jsonFileUrl) => this.read(jsonFileUrl));

  Promise.all(promises).then((values) => {
    const result = {};
    values.forEach((value) => result[value.key] = value);

    callback(result);
  }).catch((error) => console.error(error));
}
```
```javascript
read(jsonFileUrl) {
  const promise = new Promise((resolve, reject) => {
    this.readJson(jsonFileUrl).then((rawResult) => {
      const jsonResult = JSON.parse(rawResult);
      const [key, imagePath] = this.getImageData(jsonFileUrl, jsonResult);
      this.readImage(imagePath).then((imageResult) => {
        resolve({ jsonResult, imageResult, key });
      }).catch((error) => reject(error));
    }).catch((error) => reject(error));
  });

  return promise;
}
```
```javascript
readJson(jsonFileUrl) {
  const promise = new Promise((resolve, reject) => {
    const rawFile = new XMLHttpRequest();
    rawFile.overrideMimeType("application/json");
    rawFile.open("GET", jsonFileUrl, true);
    rawFile.onreadystatechange = () => {
      if (rawFile.readyState === 4) {
        if (rawFile.status == "200") {
          resolve(rawFile.responseText);
        } else {
          reject(`Fail to read ${jsonFileUrl}`);
        }
      }
    }
    rawFile.send(null);
  });

  return promise;
}
```
```javascript
readImage(imageFileUrl) {
  const promise = new Promise((resolve, reject) => {
    const image = new Image();
    image.onload = () => resolve(image);
    image.onerror = () => reject(`Fail to read ${imageFileUrl}`);
    image.src = imageFileUrl;
  });

  return promise;
}
```

Modo de uso
```javascript
const loader = new Loader(files);
loader.readAll((result) => {
  // result contiene la imagen y el json parseado
};
```

ejemplo completo [aqui](https://github.com/MisterriiXL/javascript-examples/blob/main/loader-files.js)

## Ejemplo
