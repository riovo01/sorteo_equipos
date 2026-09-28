# La Caimanera · herramientas

Dos herramientas web (PWA, sin build) para el canal/comunidad de fútbol
La Caimanera, con navegación entre ambas:

- **`index.html` — Sorteador de equipos.** Cargás los jugadores, marcás
  capitanes / grupos / separaciones y niveles, y la app reparte los
  equipos y genera una imagen lista para WhatsApp o redes.
- **`generador_miniaturas.html` — Generador de miniaturas de YouTube.**
  Textos editables con color propio, foto de fondo, 3 plantillas, formato
  16:9 y 1:1, badge, y descarga/copia del PNG. Acepta `?t1..t5` por
  querystring (el botón "Miniatura" del sorteador la abre con la fecha y
  la cancha del partido ya cargadas).

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
├── index.html              Sorteador: HTML + CSS + JS + imágenes en base64
├── generador_miniaturas.html  Generador de miniaturas (mismo criterio)
├── sw.js                   Service worker (offline, cache-first para assets,
│                           network-first para el HTML, auto-actualización)
├── manifest.webmanifest    Manifiesto PWA
├── logo2.png               Logo horizontal del encabezado (único asset por URL)
└── assets/
    ├── icons/              Íconos de la interfaz del sorteador, servidos por URL
    │                       (NO van incrustados en base64, son <img> normales):
    │       ├── capitan.png            botón/etiqueta de capitán
    │       ├── guante.png             botón/etiqueta de portero
    │       ├── pago-si.png/pago-no.png  botón/etiqueta de "pagó / no pagó"
    │       └── camiseta-{color}.png   swatch de color por equipo (uno por
    │                                   cada color de COLORS: rojo, azul,
    │                                   verde, naranja, negro, blanco)
    └── source/             Arte fuente que va INCRUSTADO en index.html como
                            base64 (no se sirve por URL):
        ├── LOGO.png             escudo del encabezado de la imagen generada
        ├── partido-de-futbol.png  título "PARTIDO DE FÚTBOL"
        ├── nos_vemos.png          cartel "¡NOS VEMOS EN LA CANCHA!"
        ├── balon.png              balón del rincón inferior derecho
        ├── arqueria.png           arco del rincón inferior izquierdo
        ├── calendario.png, reloj.png, ubicacion.png, cancha.png  íconos de
        │                          la franja de datos del partido (imagen
        │                          clásica y cartel nocturno)
        ├── pago-si.png, pago-no.png  mismos íconos que assets/icons/ pero
        │                          en el tamaño usado dentro del canvas
        ├── fondo-cancha.jpg      fondo del cartel alternativo "Cancha nocturna"
        │                          (botón "Cartel": foto de una cancha al
        │                          atardecer con un balón)
        ├── fondo-titulo.png      pincelada de fondo del título (día/horario)
        │                          del cartel "Cancha nocturna"
        ├── advertencia.png       ícono del aviso "IMPORTANTE" del cartel
        │                          "Cancha nocturna" (el otro lado del aviso
        │                          es un QR al canal, generado a mano igual
        │                          que en la imagen clásica)
        └── fonts/
            └── a-love-of-thunder.ttf  tipografía del título del cartel
                                    "Cancha nocturna" (día/horario)
```

Los íconos de `assets/icons/` que también aparecen dibujados en la imagen
generada (capitán, guante, camisetas, pago-sí/no) tienen una copia aparte
en `assets/source/` incrustada en base64: la versión servida por URL es
para el DOM (botones de la interfaz), y la de `source/` es la que se
dibuja en el `<canvas>`, porque `ctx.drawImage` de una imagen cargada por
URL "contamina" el canvas al abrir `index.html` con doble clic (sin
servidor, sin CORS) y no se puede exportar. Si actualizás uno de estos
íconos, hay que reemplazarlo en los dos lugares (y regenerar el base64).

### Sobre las imágenes embebidas

Para que la imagen generada se pueda compartir aun abriendo el archivo con
doble clic (sin servidor, sin CORS), el escudo, el título, los carteles, el
balón, el arco y la foto de fondo del cartel nocturno están incrustados en
`index.html` como `data:` URI en base64 (constantes `*_DATA_URI_B64`).

Si cambiás un arte en `assets/source/`, hay que **volver a generar el
base64** y reemplazar la constante correspondiente en `index.html`
(normalmente reescalado: p. ej. el escudo a 320×320, el balón/arco a
~560–760 px de ancho).

La tipografía del título del cartel nocturno ("A Love of Thunder", de S.
John Ross / Cumberland Games) también va incrustada en base64, como
`@font-face` en el `<style>` — es freeware de uso **no comercial**; si el
canal le da un uso público/comercial más allá de esto, revisar la
licencia en `assets/source/fonts/`. Como ese `@font-face` no lo usa
ningún elemento del HTML, el navegador no la carga solo: `drawResultImageNight`
la pide a mano con `document.fonts.load(...)` antes de dibujar el título
en el canvas.

## Funcionalidades

- Lista de jugadores por nombre (uno por línea o separados por coma).
- Capitanes, porteros, y quién pagó o no (por defecto todos figuran como
  pagados); grupos que van juntos y parejas que van separadas.
- Nivel por jugador (1–7); el reparto busca el nivel total más parejo.
- 2 o más equipos, con colores.
- Datos del partido: fecha, hora, cancha y número.
- Mover jugadores entre equipos después del sorteo.
- Imagen del resultado con el branding de La Caimanera + QR al canal de
  YouTube ("Imagen"), o cartel alternativo "Cancha nocturna" con foto de
  fondo, banner de día/horario y aviso de puntualidad ("Cartel"); ambos se
  comparten con la hoja del sistema y, si falla, se descargan.
- Guarda el estado en `localStorage`.
- Funciona offline (PWA instalable).

## Deploy

Sitio estático: se publica tal cual con GitHub Pages (rama `main`, carpeta
raíz) o cualquier hosting de archivos estáticos.
