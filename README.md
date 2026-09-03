# Sorteo de equipos · La Caimanera

App web (PWA) para armar equipos de fútbol de forma pareja y compartir el
resultado como imagen. Pensada para la previa del partido: cargás los
jugadores, marcás capitanes / grupos / separaciones y niveles, y la app
reparte los equipos y genera una imagen lista para mandar por WhatsApp o
subir a redes.

## Cómo correr

Es una sola página sin build ni dependencias:

- **Rápido:** abrí `index.html` con doble clic en el navegador.
- **Como servidor** (recomendado para probar el service worker, la PWA y
  el compartir de imágenes):

  ```sh
  npx serve .
  # o
  python -m http.server 8080
  ```

  y entrá a `http://localhost:8080`.

> El registro del service worker y el `manifest` solo se activan sobre
> `http(s)`; al abrir con `file://` se omiten a propósito.

## Estructura

```
.
├── index.html              App completa: HTML + CSS + JS + imágenes en base64
├── sw.js                   Service worker (offline, cache-first para assets,
│                           network-first para el HTML, auto-actualización)
├── manifest.webmanifest    Manifiesto PWA
├── logo2.png               Logo horizontal del encabezado (único asset por URL)
└── assets/
    └── source/             Arte fuente que va INCRUSTADO en index.html como
                            base64 (no se sirve por URL):
        ├── LOGO.png             escudo del encabezado de la imagen generada
        ├── partido-de-futbol.png  título "PARTIDO DE FÚTBOL"
        ├── nos_vemos.png          cartel "¡NOS VEMOS EN LA CANCHA!"
        ├── balon.png              balón del rincón inferior derecho
        └── arqueria.png           arco del rincón inferior izquierdo
```

### Sobre las imágenes embebidas

Para que la imagen generada se pueda compartir aun abriendo el archivo con
doble clic (sin servidor, sin CORS), el escudo, el título, los carteles, el
balón y el arco están incrustados en `index.html` como `data:` URI en
base64 (constantes `*_DATA_URI_B64`).

Si cambiás un arte en `assets/source/`, hay que **volver a generar el
base64** y reemplazar la constante correspondiente en `index.html`
(normalmente reescalado: p. ej. el escudo a 320×320, el balón/arco a
~560–760 px de ancho).

## Funcionalidades

- Lista de jugadores por nombre (uno por línea o separados por coma).
- Capitanes, porteros, grupos que van juntos y parejas que van separadas.
- Nivel por jugador (1–7); el reparto busca el nivel total más parejo.
- 2 o más equipos, con colores.
- Datos del partido: fecha, hora, cancha y número.
- Mover jugadores entre equipos después del sorteo.
- Imagen del resultado con el branding de La Caimanera + QR al canal de
  YouTube; se comparte con la hoja del sistema y, si falla, se descarga.
- Guarda el estado en `localStorage`.
- Funciona offline (PWA instalable).

## Deploy

Sitio estático: se publica tal cual con GitHub Pages (rama `main`, carpeta
raíz) o cualquier hosting de archivos estáticos.
