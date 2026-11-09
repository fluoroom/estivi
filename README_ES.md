# Estivi

[English](README.md) · **Español**

**SSTV, con agenda.**

La mayoría del software de SSTV manda una imagen cuando se lo pedís. Estivi corre un canal. Armás una semana —programas que se repiten, bloques de una sola fecha, las imágenes y las cortinas que van adentro— y el transmisor acciona la radio y la pone al aire, con su indicativo y todo. Del otro lado, un receptor en el navegador convierte el audio de la radio en imágenes y un calendario, para que quien escucha pueda seguir la programación mientras sale.

La televisión de barrido lento (SSTV) manda imágenes fijas por un canal de voz de banda angosta, una línea por vez. Estivi las trata como programación.

<p align="center">
  <img src="https://raw.githubusercontent.com/fluoroom/estivi-tx/main/docs/screenshots/main.png" alt="Transmisor de Estivi: la grilla semanal, la próxima transmisión y el bloque seleccionado" width="920">
</p>

<p align="center"><em>El transmisor: la grilla semanal, la próxima transmisión y la línea de tiempo del bloque seleccionado.</em></p>

`Transmisor 0.2.2` · `Linux, Windows, macOS` · `Rust + Slint` · `JavaScript + Web Audio`

## Los tres repositorios

| Repositorio | Rol | Stack |
|---|---|---|
| **[estivi-tx](https://github.com/fluoroom/estivi-tx)** | La estación. Una app de escritorio que sostiene el canal, dibuja la grilla, renderiza todos los modos SSTV, acciona la radio, responde CW y DTMF, registra cada transmisión y manda una imagen suelta fuera de la grilla. | Rust, Slint |
| **[estivi-rx](https://github.com/fluoroom/estivi-rx)** | El receptor. Una página web que demodula el audio de la radio en el navegador, decodifica las imágenes y la guía AX.25, y mantiene una línea de tiempo, un calendario y PNG guardados. Se instala como app y se acuerda de todo entre visitas. | JavaScript, Web Audio, nginx |
| **[estivi-webpage](https://github.com/fluoroom/estivi-webpage)** | La puerta de entrada. La página pública del proyecto, en inglés y español: un receptor simulado, la grilla semanal de ejemplo, un explorador de modos y una galería de las dos puntas. | HTML, CSS, JavaScript, nginx |

Este repositorio es el paraguas del proyecto. Fija los tres proyectos como submódulos en commits conocidos y no tiene código propio; cada submódulo mantiene su historia, sus releases y su documentación.

## Cómo encaja

```text
 .estivi canal ──▶ estivi-tx ──▶ radio ═══ aire ═══▶ radio ──▶ estivi-rx
   tu agenda        audio + PTT     SSTV + AX.25    micrófono   imágenes y guía
```

Entre las dos puntas no hay ningún enlace privado. Todo lo que sabe el receptor llega como sonido: imágenes SSTV con sus cabeceras VIS y tramas UI de AX.25 a 1200 baudios dirigidas a `ESTIVI`, que llevan la guía de la estación y nombran cada pieza. Quien se pierde la guía igual recibe las imágenes; el calendario se va completando a medida que llegan las tramas.

## Probalo

### Transmitir

Descargá la [última versión](https://github.com/fluoroom/estivi-tx/releases/latest) —Linux x86_64, Windows x86_64/i686 o macOS universal—, descomprimila y ejecutá `estivi`. Abrí [`ejemplo/canal-demo.estivi`](https://github.com/fluoroom/estivi-tx/blob/main/ejemplo/canal-demo.estivi) para ver un canal completo: dos programas, bloques guardados, activaciones, un zócalo, un QR y una capa de clima oculta. Todas las imágenes que usa están dentro de `ejemplo/`, así que la muestra corre en cualquier máquina.

Configurá tu indicativo, la salida de audio y el dispositivo de PTT, y después **Iniciar**. ¿No tenés una radio a mano? Exportá cualquier rango a FLAC.

### Recibir

El receptor público está en **[estivi-rx.lu1aqy.radio](https://estivi-rx.lu1aqy.radio)**. Abrilo, permití el micrófono y apuntalo al parlante de tu receptor, o pasale el audio de línea del receptor. La decodificación ocurre en la página; el servidor solo sirve archivos estáticos.

<p align="center">
  <img src="https://raw.githubusercontent.com/fluoroom/estivi-rx/main/docs/screenshots/main.png" alt="Estivi RX: una imagen de atardecer recibida en el visor, la línea de tiempo debajo y la fila de horas de la estación" width="920">
</p>

<p align="center"><em>El receptor: el visor, la línea de tiempo y la fila de horas de la estación.</em></p>

Para correr tu propia copia:

```sh
git clone --recurse-submodules https://github.com/fluoroom/estivi.git
cd estivi/estivi-rx
docker compose up -d --build
# abrí http://localhost:8086
```

Los navegadores solo dan el micrófono por HTTPS o en localhost, así que poné un terminador TLS delante cuando lo despliegues. Después de la primera visita, el receptor también abre sin conexión.

Si clonaste sin `--recurse-submodules`, completá los submódulos con `git submodule update --init --recursive`.

### La página web

La página web es la puerta de entrada: una decodificación simulada en el receptor, la grilla semanal de ejemplo, el explorador de modos SSTV y capturas de las dos puntas, en inglés y español. Es HTML, CSS y JavaScript puros, servida por nginx:

```sh
cd estivi-webpage
docker compose up -d --build
# abrí http://localhost:8087
```

La página en inglés está en `/` y la española en `/es/`.

## Qué hace el transmisor

- **Una semana, no una cola.** Cada día de la semana tiene su propio inicio, fin e intervalo. El lunes puede durar quince minutos; el sábado, horas. Se arma una vez y listo.
- **Bloques para lo demás.** Una fecha, una tarjeta: un evento especial, una ronda, un concurso. Movelo, salteá ese día o borralo.
- **Imágenes con cortinas.** La bienvenida y la despedida salen en el primer y último turno; las cortinas previas y posteriores viajan con su imagen. Cada imagen puede heredar su modo o elegir otro.
- **Instantánea.** Mandá una imagen fuera de la grilla, con sus propias cortinas, modo y nivel de gráficos.
- **Le contesta a la radio.** Las activaciones por CW y DTMF ejecutan una imagen, un comando, una respuesta de API o una secuencia de acciones.
- **Gráficos sin otra herramienta.** Capas de texto, imagen y QR por canal, programa o bloque, sobre la próxima imagen al aire o sobre un tablero de transparencia, barras de color, una grilla o una rampa de grises.
- **En vivo y Siguiente.** Mirá la imagen llenarse línea por línea mientras sale, y qué viene después con su hora y su nombre de archivo.
- **Un solo ejecutable.** Renderizado por software, unos 50 MB de RAM, siete temas, inglés o español y un registro CSV de cada transmisión.

## Qué hace el receptor

- **Decodifica en la página.** El audio de la radio pasa por una etapa de ganancia a un AudioWorklet; AX.25 y SSTV corren enteros en el navegador, sin decodificador en el servidor.
- **Lee la guía.** Las tramas AX.25 nombran la estación y la programación, así que el receptor arma con ellas un calendario de 30 días y una fila de horas.
- **Retiene lo que escucha.** El visor muestra las líneas a medida que llegan; las imágenes terminadas van a una línea de tiempo, se pueden guardar como PNG y sobreviven a una recarga en IndexedDB, hasta 40.
- **Sigue el reloj.** La fila de horas arranca en la hora en curso, una línea amarilla viaja con el reloj local y una marca fina se posa en cada imagen guardada.
- **Aguanta una mala señal.** Un desvanecimiento corto no tira la imagen: el decodificador espera el 30% del tiempo de imagen del modo y retoma donde vuelve la señal.
- **Sigue andando.** Se instala como app en navegadores Chromium, se agrega a la pantalla de inicio en iPhone y iPad, y un service worker sirve una copia en caché después de la primera visita.
- **A tu gusto.** Oscuro o claro, inglés o español, recordados entre recargas junto con las imágenes y la guía.

## Al aire

- **SSTV.** 22 modos — Martin M1–M4, Scottie S1/S2/DX, Robot 24/36/72, Robot 12 color, Robot B/W 8/12/24, PD 50, 90, 120, 160, 180, 240 y 290, más YoniQ. Cada pieza va precedida por su VIS.
- **AX.25.** Bell 202 a 1200 baudios, tramas UI dirigidas a `ESTIVI`. La guía lleva `FREQ`, `CALL`, `NAME`, `RANGE`, `DAYS`, `DAYSLEFT`, `NEXT` y `COMMENT`; las tramas de modo anuncian cada pieza, como en `ESTIVI MODE R36 BUMPER PRE`.
- **Identificación.** CW opcional antes y después de las imágenes, con el mismo indicativo. El PTT es serie o CAT por `rigctld`.
- **Audio.** Sintetizado a 11025, 22050, 44100 o 48000 Hz, y exportado como FLAC.

## Desarrollo

### Los submódulos

Cada submódulo es un repositorio normal, y los tres siguen `main`. Para actualizar los tres al `main` actual:

```sh
git submodule update --remote --merge
git add estivi-tx estivi-rx estivi-webpage
git commit -m "Update submodules"
```

El trabajo se hace en el submódulo que lo contiene; un cambio en el paraguas solo mueve la referencia.

### Transmisor

Rust estable 1.85 o superior. Linux necesita ALSA, fontconfig y GTK 3; macOS, las herramientas de línea de comandos de Xcode; Windows, el toolchain MSVC.

```sh
cd estivi-tx
cargo build --release   # target/release/estivi
cargo test
```

El [manual](https://github.com/fluoroom/estivi-tx/blob/main/docs/MANUAL.md) cubre el orden de un turno, cada comportamiento de la ventana, los formatos de archivo, importación y exportación, y lo que se verificó al aire. Está en inglés.

### Receptor

No hay nada que compilar: `www/` es la página y el archivo compose la sirve con nginx. Las pruebas del decodificador corren con el runner integrado de Node:

```sh
cd estivi-rx
npm test
```

## Documentación

- [README del transmisor](https://github.com/fluoroom/estivi-tx#readme) — funcionalidades, capturas, releases y notas de compilación. También en [inglés](https://github.com/fluoroom/estivi-tx/blob/main/README.md).
- [Manual del transmisor](https://github.com/fluoroom/estivi-tx/blob/main/docs/MANUAL.md) — la versión larga, en inglés.
- [Interfaz AX.25](https://github.com/fluoroom/estivi-tx/blob/main/AX25.md) — los bytes y el texto exactos que lee un cliente.
- [README del receptor](https://github.com/fluoroom/estivi-rx#readme) — el recorrido de la página, el decodificador y el almacenamiento. También en [inglés](https://github.com/fluoroom/estivi-rx/blob/main/README.md).
- [README de la página web](https://github.com/fluoroom/estivi-webpage#readme) — el recorrido, el selector de idioma, el despliegue y la política de nginx.
- [Este README en inglés](README.md).
