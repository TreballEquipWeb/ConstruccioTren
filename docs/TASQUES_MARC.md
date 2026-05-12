# TASQUES_MARC — Canvis implementats

## Issues implementades

- **#16 — Sistema de input:** detecció de clics del ratolí sobre el canvas de Phaser, identificant fila/columna de la casella clicada.
- **#19 — Colocar rail, talar, destruir:** connexió del clic amb les accions `talarArbre`, `destruirObstacle` i `colocarRailEn` segons el tipus de casella.

---

## Fitxers modificats

### `src/scenes/PlayScene.js`

**Canvis:**

- Eliminades les accions de mostra de `create()` (`talarArbre`, `destruirObstacle`, `colocarRailEn` per codi).
- Afegides dues constants de mòdul:
  - `MIDA_CASELLA = 80` — mida en píxels de cada casella del grid.
  - `COLORS_CASELLA` — mapa de tipus de casella a colors hexadecimals per al renderitzat.
- A `create()`:
  - Càlcul de `this.offsetX` i `this.offsetY` per centrar el grid al canvas (800×600).
  - Creació de `this.graphics` (objecte Phaser Graphics per dibuixar).
  - Crida inicial a `this.dibuixarMapa()`.
  - Registre del listener `pointerdown` sobre `this.input` → `this.onClic`.
- Afegit mètode `dibuixarMapa()`:
  - Neteja el graphics (`clear()`).
  - Itera totes les caselles del mapa i dibuixa un rectangle de color + border negre per a cadascuna.
- Afegit mètode `onClic(pointer)`:
  - Ignora el clic si `this.joc.estat !== 'jugant'`.
  - Converteix les coordenades del punter a fila/columna del grid.
  - Ignora clics fora dels límits del mapa.
  - Executa l'acció corresponent segons `casella.tipus`:
    - `BOSC` → `this.joc.jugador.talarArbre(casella)`
    - `OBSTACLE` → `this.joc.jugador.destruirObstacle(casella)`
    - `PLA` → `this.joc.colocarRailEn(fila, columna)`
  - Crida `this.dibuixarMapa()` per actualitzar el visual.

---

## Fitxers creats

### `TASQUES_MARC.md` (aquest fitxer)

---

## Fitxers actualitzats

### `DOCUMENTACIO.md`

- Secció 6: actualitzat "Què hi ha fet" amb #16 i #19, taula de colors, i eliminat "Renderitzat al canvas" i "Interacció d'usuari" de "Què falta".
- Secció 8: actualitzat l'historial recent.

---

## Classes NO modificades

Tal com exigeixen les regles: `Casella`, `Mapa`, `Jugador`, `Nivell`, `SistemaEstrelles`, `Joc` no han estat tocades.

---

## Reorganització de `src/` (sense canvi de lògica)

### Fitxers creats

| Fitxer | Origen |
|--------|--------|
| `src/styles/base.css` | Extret de `src/style.css` (reset + variables CSS) |
| `src/styles/layout.css` | Extret de `src/style.css` (estructura general) |
| `src/styles/canvas.css` | Extret de `src/style.css` (estils del canvas) |
| `src/styles/main.css` | Nou — `@import` dels 3 CSS anteriors |
| `src/constants/tiposCasella.js` | Mogut des de `src/classes/tiposCasella.js` (contingut idèntic) |
| `src/constants/colors.js` | Extret de `src/scenes/PlayScene.js` (`COLORS_CASELLA`) |
| `src/config/nivells.js` | Extret de `src/scenes/PlayScene.js` (config `NIVELL_PROVA`) |
| `src/assets/.gitkeep` | Nou — carpeta reservada per a recursos |

### Fitxers eliminats

| Fitxer | Motiu |
|--------|-------|
| `src/style.css` | Dividit en `src/styles/` |
| `src/classes/tiposCasella.js` | Mogut a `src/constants/tiposCasella.js` |

### Fitxers amb imports actualitzats

| Fitxer | Canvi |
|--------|-------|
| `src/main.js` | `./style.css` → `./styles/main.css` |
| `src/classes/Casella.js` | `./tiposCasella.js` → `../constants/tiposCasella.js` |
| `src/classes/Mapa.js` | `./tiposCasella.js` → `../constants/tiposCasella.js` |
| `src/classes/Jugador.js` | `./tiposCasella.js` → `../constants/tiposCasella.js` |
| `src/classes/Joc.js` | `./tiposCasella.js` → `../constants/tiposCasella.js` |
| `src/classes/index.js` | `./tiposCasella.js` → `../constants/tiposCasella.js` |
| `src/scenes/PlayScene.js` | Afegits imports de `COLORS_CASELLA` i `NIVELL_PROVA`; eliminades definicions inline |

### `docs/DOCUMENTACIO.md`

- Secció 2: estructura nova de `src/` amb `styles/`, `constants/`, `config/`, `assets/`.
- Secció 3: flux d'arrencada actualitzat (referència a `styles/main.css`).
- Secció 4.1: ruta canviada de `classes/tiposCasella.js` a `constants/tiposCasella.js`.
- Secció 6: snapshot actualitzat (config viu a `config/nivells.js`).
- Secció 7: nota del barrel actualitzada.
- Secció 8: historial afegit.
