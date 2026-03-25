# PokéPrint 🃏⚡

**Generador de PDF para imprimir mazos de Pokémon TCG — 100% en el navegador, sin servidor.**

Sube tu lista de cartas, genera dos PDFs listos para imprimir en dúplex y descárgatelos. Nada que instalar, nada que ejecutar en la terminal.

---

## ✨ Características

- **Sin servidor** — todo ocurre en el browser via `pdf-lib`
- **Descarga automática del reverso** — usa la imagen oficial de Bulbapedia al abrir la página
- **Reverso personalizado** — arrastra tu propio PNG/JPG para sobreescribir el oficial
- **Grilla 3×3** con separación configurable entre cartas (`CARD_GAP`)
- **Espejado automático de columnas** para impresión dúplex en eje largo
- **Última hoja del reverso** con exactamente las mismas cartas que la última hoja del frente
- **Página de calibración** incluida — imprime antes de gastar tinta en las cartas
- **Formatos A4, Carta o ambos** a elegir
- **API key opcional** — funciona sin key, pero con key de `pokemontcg.io` va más rápido

---

## 🚀 Uso

### 1. Abrir la herramienta

Abre `pokeprint.html` directamente en cualquier navegador moderno (Chrome, Firefox, Edge, Safari). No necesita servidor local.

> Para publicarla online, sube el archivo a cualquier hosting estático: **GitHub Pages**, **Netlify**, **Vercel**, etc.

### 2. Pegar la lista del mazo

El formato esperado es el estándar de exportación de clientes como **PTCGL** o **Limitless TCG**:

```
4 Iron Crown ex TEF 81
4 Munkidori TWM 95
2 Fezandipiti ex TWM 92
4 Bibarel BRS 121
2 Bidoof BRS 120
4 Comfey LOR 79
```

Patrón de cada línea: `CANTIDAD NOMBRE_CARTA SET NÚMERO`

### 3. Reverso

Al cargar la página se descarga automáticamente el reverso oficial inglés desde Bulbapedia. Si quieres usar uno personalizado (p.ej. un reverso proxy), arrastra tu imagen encima del dropzone.

### 4. Configuración (opcional)

| Campo | Descripción |
|---|---|
| **API Key** | Clave de [pokemontcg.io](https://dev.pokemontcg.io) — gratis, aumenta el rate limit |
| **Formato** | A4 · Carta · o ambos simultáneamente |

La API key se guarda en `localStorage` para no tener que reescribirla.

### 5. Generar y descargar

Haz clic en **⚡ GENERAR PDFs**. La herramienta:

1. Parsea tu lista y deduplica las cartas únicas
2. Consulta `pokemontcg.io` para obtener la URL de imagen de cada carta
3. Descarga las imágenes del CDN de Pokémon TCG
4. Genera los PDFs en memoria y los pone disponibles para descarga

Por cada formato seleccionado obtienes **dos archivos**:

| Archivo | Contenido |
|---|---|
| `print_deck_A4.pdf` | Frentes + reversos espejados, listo para dúplex |
| `calibracion_A4.pdf` | Página de prueba con flechas y rayado — imprime primero |

---

## 🖨️ Instrucciones de impresión

### Paso previo: calibración

1. Imprime **solo** `calibracion_A4.pdf`
2. Da vuelta el papel en el eje **largo** (como abrir un libro horizontalmente)
3. Imprime la **hoja 2** del mazo sobre ese papel
4. Verifica que los bordes y la flecha ↑ coincidan exactamente con los slots del frente

Si hay desalineación, ajusta los márgenes de tu impresora hasta que cuadren.

### Impresión final

- Modo: **Dúplex / Doble cara**
- Voltear en: **Eje largo** (`Flip on Long Edge`)
- Los reversos ya están espejados automáticamente — no toques ninguna opción de espejo en la impresora

---

## 🔧 Equivalente en Python

Este proyecto nació como script Python (`generate_print_deck.py`) para uso local. La web replica exactamente la misma lógica:

| Python | Web |
|---|---|
| `reportlab` + puntos PDF | `pdf-lib` + puntos PDF |
| `CARD_W/H` en mm | Mismas dimensiones convertidas a pt |
| `CARD_GAP` entre cartas | Idéntico |
| Espejado de columnas en reverso | Idéntico |
| Última hoja del reverso = última del frente | Idéntico |
| Carpeta con imágenes locales | API `pokemontcg.io` + CDN |
| `BACK.png` local | Bulbapedia CDN + override manual |

Para uso por lote o integración en scripts, el `.py` sigue siendo la opción recomendada. Para uso puntual sin instalación, usa la web.

---

## 📦 Dependencias

La web usa estas librerías vía CDN, sin `npm install`:

| Librería | Versión | Uso |
|---|---|---|
| [pdf-lib](https://pdf-lib.js.org/) | 1.17.1 | Generación de PDFs en el browser |
| [Google Fonts](https://fonts.google.com/) | — | Bangers, Outfit, DM Mono |

**API externa:**

| Servicio | URL | Auth |
|---|---|---|
| Pokémon TCG API | `api.pokemontcg.io/v2` | Sin key (limitado) o con key gratis |
| Pokémon TCG CDN | `images.pokemontcg.io` | Pública |
| Bulbagarden CDN | `cdn2.bulbagarden.net` | Pública |
| corsproxy.io | `corsproxy.io` | Pública (fallback CORS) |

---

## ⚙️ Parámetros técnicos de las cartas

| Parámetro | Valor |
|---|---|
| Ancho de carta | 63.5 mm (2.5 in) |
| Alto de carta | 88.9 mm (3.5 in) |
| Separación entre cartas | 2 mm |
| Cartas por página | 3 × 3 = 9 |
| Página A4 | 210 × 297 mm |
| Página Carta | 215.9 × 279.4 mm |

Para modificar la separación entre cartas, edita la constante `CARD_GAP` en el JS (web) o en el `.py` (script):

```js
// pokeprint.html
const CARD_GAP = 2 * MM;   // cambia este valor
```

---

## 🐛 Problemas comunes

**"IMAGEN NO ENCONTRADA" en el PDF**
El código de set o número no coincide con los de `pokemontcg.io`. Verifica el set code exportado desde PTCGL — algunos sets tienen códigos distintos en la API (ej. `SVI` vs `sv1`).

**La API va muy lenta sin key**
Sin API key hay un rate limit estricto. Regístrate gratis en [dev.pokemontcg.io](https://dev.pokemontcg.io), la key te da ~1000 req/día.

**El reverso oficial no carga automáticamente**
Puede ser un bloqueo de CORS o de red. Descarga manualmente `Cardback.jpg` desde [Bulbapedia](https://bulbapedia.bulbagarden.net/wiki/File:Cardback.jpg) y arrástralo al dropzone.

**Las cartas no alinean al recortar**
Sigue el proceso de calibración antes de imprimir el mazo completo. Cada impresora tiene márgenes físicos distintos.

---

## 📄 Licencia

Imágenes de cartas © The Pokémon Company. Este proyecto es una herramienta personal sin fines comerciales.
