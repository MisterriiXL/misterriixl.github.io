---
layout: post
title: Múltiples archivos en JavaScript
tags:
- javascript
date: 2022-08-09 23:49 -0400
---
Leer un conjunto de archivos en formato **json**, cada uno de estos archivos contienen un parámetro con el nombre de una imagen, en este caso un png. La idea es que al terminar de cargar todos los archivos invoque una función predefinida (callback).



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

Originalmente pensé que cada vez que terminara de cargar un archivo, ya sea el json o el png hacer un callback, pero a medida que los archivos aumentaban el proceso se ponía mas complejo y engorroso, por lo tanto la opción que encontré es hacer estas llamadas asíncronas a través de **promise**, ya que estos están diseñados para saber si un proceso asíncrono finalizo con éxito o con error.

Para la solución implemente 4 métodos:
 - Método para leer un archivo json.
 - Método para leer una imagen.
 - Método que relaciona el json con su imagen.
 - Método que lee la lista completa de archivos.



### Método que lee un archivo json

Este método recibe como parámetro la url del archivo json, una vez que el archivo se cargue resolvemos la promesa, en caso de fallo rechazamos.
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



### Método que lee una imagen

Este método recibe como parámetro la **url** de la imagen, una vez que la imagen se carga resolvemos la promesa o en caso de fallo rechazamos.
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



### Método que relaciona el json con su imagen

Este método recibe como parámetro la url del json, una vez que se lee correctamente el archivo json obtenemos la información de la imagen para carga el contenido de esta.

> El método **getImageData** entrega la dirección completa de la imagen mas una key que se utilizara como identificador.

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


### Método que lee la lista completa de archivos

Este método recibe como parámetro la funcionan callback que se invocara una vez que se resuelvan todas las promesas creados.
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



## Modo de uso

El ejemplo completo lo puedes encontrar [aquí](https://github.com/MisterriiXL/javascript-examples/blob/main/loader-files.js) , ya que solo se describió los métodos más relevantes.

```javascript
const loader = new Loader(files);
loader.readAll((result) => {
  // result contiene la imagen y el json parseado
};
```

Cualquier duda o sugerencia siempre es Bienvenida.
