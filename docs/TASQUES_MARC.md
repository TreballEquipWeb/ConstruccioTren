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
