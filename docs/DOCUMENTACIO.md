# ConstruccioTren — Documentació del projecte

Document a banda del `README.md` amb l'estat actual del projecte i com funciona.

---

## 1. Resum general

**ConstruccioTren** és un joc/prototip web on el jugador ha de construir un camí de rails entre una casella d'inici (`INICI`) i una de meta (`META`) sobre un mapa quadriculat. Per fer-ho disposa de recursos limitats (rails, tales d'arbres i destruccions d'obstacles) i ha de gestionar-los per arribar al final.

- **Llenguatge:** JavaScript (ES Modules)
- **Motor de joc:** [Phaser 4](https://phaser.io/) (`phaser ^4.1.0`)
- **Bundler / dev server:** [Vite](https://vitejs.dev/) (`vite ^8.0.10`)
- **Gestor de paquets:** pnpm (existeix `pnpm-lock.yaml`)
- **Branca actual:** `main`

---

## 2. Estructura del projecte

```
ConstruccioTren/
├── index.html              # HTML d'entrada, conté el contenidor #gameCanvas
├── package.json            # Scripts (dev/build/preview) i dependencies
├── pnpm-lock.yaml
├── README.md               # Actualment només el títol
├── .gitignore
├── public/
│   ├── favicon.svg
│   └── icons.svg
├── docs/
│   └── DOCUMENTACIO.md     # Aquest fitxer
└── src/
    ├── main.js             # Punt d'entrada: configura Phaser i carrega PlayScene
    ├── styles/
    │   ├── main.css        # @import dels altres 3
    │   ├── base.css        # Reset + variables CSS
    │   ├── layout.css      # Estructura general (body, header, main…)
    │   └── canvas.css      # Estils del canvas de Phaser
    ├── scenes/
    │   └── PlayScene.js    # Escena de Phaser que orquestra el joc
    ├── classes/
    │   ├── index.js            # Re-exporta totes les classes i TIPOS_CASILLA
    │   ├── Casella.js          # Una cel·la del mapa
    │   ├── Mapa.js             # Matriu de Caselles + lògica de camí
    │   ├── Jugador.js          # Recursos del jugador i accions
    │   ├── Nivell.js           # Configuració d'un nivell
    │   ├── SistemaEstrelles.js # Càlcul d'estrelles segons eficiència
    │   └── Joc.js              # Orquestrador principal de l'estat del joc
    ├── constants/
    │   ├── tiposCasella.js     # Constants TIPOS_CASILLA
    │   ├── colors.js           # Constants COLORS_CASELLA (colors per tipus de casella)
    │   └── ui.js               # Constants UI_COLORS, UI_DEPTH, UI_STYLES
    ├── config/
    │   └── nivells.js          # Configuració del nivell de prova (NIVELL_PROVA)
    ├── ui/
    │   └── HUD.js              # Panel HUD amb comptadors de recursos
    └── assets/                 # Recursos estàtics (carpeta reservada)
```

---

## 3. Com s'executa

Scripts definits a `package.json`:

| Script           | Comanda          | Descripció                         |
|------------------|------------------|------------------------------------|
| `pnpm dev`       | `vite`           | Servidor de desenvolupament local  |
| `pnpm build`     | `vite build`     | Build de producció (a `dist/`)     |
| `pnpm preview`   | `vite preview`   | Servir el build de producció       |

Flux d'arrencada:

1. `index.html` carrega `/src/main.js` com a mòdul ES.
2. `main.js` importa `src/styles/main.css` i crea una instància de `Phaser.Game` amb resolució 800×600, físiques arcade i registra l'única escena: `PlayScene`.
3. `PlayScene.create()` instancia `Nivell` amb la config de `src/config/nivells.js`, crida `Joc.iniciarJoc(nivell)` i configura el renderitzat i el sistema d'input.

---

## 4. Model de domini

### 4.1 `TIPOS_CASILLA` (`src/constants/tiposCasella.js`)

Constants amb els tipus possibles de casella:

| Constant   | Valor       | Significat                                      |
|------------|-------------|-------------------------------------------------|
| `PLA`      | `'pla'`     | Terreny pla, construïble                        |
| `BOSC`     | `'bosc'`    | Bosc — es pot talar per obtenir un rail         |
| `OBSTACLE` | `'obstacle'`| Obstacle — es pot destruir per deixar `PLA`    |
| `AIGUA`    | `'aigua'`   | Aigua — immutable i inaccessible                |
| `RAIL`     | `'rail'`    | Rail ja col·locat                               |
| `INICI`    | `'inici'`   | Punt d'inici del recorregut                     |
| `META`     | `'meta'`    | Punt final del recorregut                       |

### 4.2 `Casella` (`src/classes/Casella.js`)

Representa una cel·la del mapa.

- **Estat:** `tipus` (un valor de `TIPOS_CASILLA`).
- **Mètodes:**
  - `canviarTipus(nouTipus)` — substitueix el tipus.
  - `esConstruible()` — `true` si el tipus és `PLA`.

### 4.3 `Mapa` (`src/classes/Mapa.js`)

Matriu de `Casella` + utilitats de cerca i validació de camí.

- **Constructor:** rep una matriu (de strings o `Casella`) i normalitza tot a `Casella`.
- **Atributs:** `caselles` (matriu) i `mida` (`{files, columnes}`).
- **Mètodes principals:**
  - `obtenirCasella(fila, columna)` — accés segur (retorna `null` si fora de rang).
  - `reiniciar(mapaInicial)` — reconstrueix el mapa des d'una matriu de tipus.
  - `trobarPosicio(tipus)` — primera ocurrència d'un tipus.
  - `contarTipus(tipus)` — quantes caselles d'aquell tipus queden (s'usa per detectar derrota).
  - `comprovarCami()` — **BFS** des de l'`INICI` fins a la `META` travessant només caselles `RAIL`/`INICI`/`META` en les 4 direccions cardinals. Retorna `true` si hi ha camí connex.

### 4.4 `Jugador` (`src/classes/Jugador.js`)

Estat de recursos i accions disponibles.

- **Atributs:** `rails`, `talesDisponibles`, `destruccionsDisponibles`.
- **Mètodes:**
  - `talarArbre(casella)` — només sobre `BOSC` i si queden tales: la converteix en `PLA`, gasta 1 tala i guanya **+1 rail**.
  - `destruirObstacle(casella)` — només sobre `OBSTACLE` i si queden destruccions: la converteix en `PLA` i gasta 1 destrucció (no dóna rails).
  - `colocarRail(casella)` — només sobre `PLA` i si té rails: la converteix en `RAIL` i resta 1 rail.

Tots tres retornen `boolean` indicant si l'acció s'ha aplicat.

### 4.5 `Nivell` (`src/classes/Nivell.js`)

Configuració d'un nivell concret.

- **Constructor (objecte d'opcions):** `nom`, `mapaInicial`, `railsInicials`, `limitsAccions: { tales, destruccions, rails }`.
- **Mètodes:**
  - `iniciarNivell()` — retorna una còpia clonada de l'estat inicial (per evitar mutacions externes).

### 4.6 `SistemaEstrelles` (`src/classes/SistemaEstrelles.js`)

Càlcul de la puntuació final en estrelles (0–3).

- **Constructor:** `criteriBase` (per defecte 3, accions ideals per a 3 estrelles).
- **Mètode `calcularEstrelles({ victoria, accionsUsades, accionsIdeals, recursosRestants })`:**
  - Sense victòria → **0** estrelles.
  - `accionsUsades ≤ accionsIdeals` **i** `recursosRestants > 0` → **3** estrelles.
  - `accionsUsades ≤ accionsIdeals + 2` → **2** estrelles.
  - Altrament → **1** estrella.

### 4.7 `Joc` (`src/classes/Joc.js`)

Classe orquestradora. Manté l'estat global del joc per nivell.

- **Atributs:** `nivellActual`, `jugador`, `mapa`, `sistemaEstrelles`, `estat` (`'inactiu' | 'jugant' | 'victoria' | 'derrota'`).
- **Mètodes:**
  - `iniciarJoc(nivell)` — comprova que `nivell` és instància de `Nivell`, crea `Mapa` i `Jugador` amb la configuració, posa `estat = 'jugant'` i retorna `this` (encadenable).
  - `colocarRailEn(fila, columna)` — embolcall que:
    1. Obté la `Casella` i delega a `jugador.colocarRail`.
    2. Si hi ha victòria (camí complet) → finalitza i retorna `{success, victoria, estrelles}`.
    3. Si no queden rails **i** ja no es poden generar més (ni `BOSC` al mapa ni tales disponibles) → finalitza com a derrota.
    4. Altrament continua la partida.
  - `comprovarVictoria()` — delega a `mapa.comprovarCami()`.
  - `finalitzarPartida(accionsUsades = 0)` — fixa `estat` segons resultat i calcula estrelles via `SistemaEstrelles`.

---

## 5. Flux d'una partida

```
Nivell (config) ──► Joc.iniciarJoc(nivell)
                         │
                         ├─► crea Mapa a partir de mapaInicial
                         └─► crea Jugador amb railsInicials i límits

Bucle de joc (manual ara mateix):
   jugador.talarArbre(casella BOSC)     → +1 rail
   jugador.destruirObstacle(casella OBSTACLE) → casella passa a PLA
   joc.colocarRailEn(fila, columna)     → consumeix 1 rail i comprova victòria/derrota

Final:
   victoria  → BFS troba camí INICI ↔ META → estrelles segons eficiència
   derrota   → no queden rails ni manera d'obtenir-ne
```

---

## 6. Estat actual (snapshot)

La config del nivell de prova viu a `src/config/nivells.js` com a `NIVELL_PROVA`:

```js
// src/config/nivells.js
export const NIVELL_PROVA = {
  nom: 'Nivell de prova',
  mapaInicial: [
    [INICI, PLA,      BOSC, PLA,  META],
    [PLA,   OBSTACLE, PLA,  BOSC, PLA ],
    [PLA,   PLA,      PLA,  PLA,  PLA ]
  ],
  railsInicials: 2,
  limitsAccions: { tales: 1, destruccions: 1, rails: 2 }
}
```

`PlayScene.create()` l'instancia amb `new Nivell(NIVELL_PROVA)`.

### Què hi ha fet

- Model de dades complet: `Casella`, `Mapa`, `Jugador`, `Nivell`, `SistemaEstrelles`, `Joc`.
- Detecció de camí amb BFS sobre caselles connectables.
- Gestió de recursos (rails, tales, destruccions) amb validacions.
- Càlcul d'estrelles 0–3 segons victòria, accions i recursos sobrants; `SistemaEstrelles` rep ara un array del camí òptim per calcular `accionsIdeals`.
- Detecció de victòria automàtica en col·locar un rail i de derrota quan ja no es poden generar més rails.
- Mètode `Mapa.reiniciar()` per reconstruir el tauler.
- Bootstrap de Phaser + Vite funcionant; escena `PlayScene` registrada.
- **#16 Sistema de input:** detecció de clics del ratolí sobre el canvas. `onClic` calcula la fila i columna a partir de les coordenades del punter i el desplaçament del grid (`offsetX`, `offsetY`, `MIDA_CASELLA`).
- **#19 Colocar rail, talar, destruir:** el clic connecta amb les accions del model: `BOSC` → `talarArbre`, `OBSTACLE` → `destruirObstacle`, `PLA` → `colocarRailEn`. Ignora clics fora del grid i quan `estat !== 'jugant'`.
- **Renderitzat bàsic al canvas:** `dibuixarMapa()` dibuixa rectangles de color per a cada casella usant `Phaser.GameObjects.Graphics`. Es crida a l'inici i cada vegada que un clic modifica l'estat.
- **Reorganització de `src/`:** separació en `styles/`, `constants/`, `config/`, `ui/` i `assets/`.
- **HUD de recursos (`src/ui/HUD.js`):** panel superior centrat que mostra en temps real els comptadors de rails, tales i destruccions del jugador. S'actualitza a cada acció.
- **Constants UI (`src/constants/ui.js`):** centralitza `UI_COLORS`, `UI_DEPTH` i `UI_STYLES` per a tota la interfície (HUD, overlays de victòria/derrota, estrelles).
- **Pantalla de resultat:** overlay semitransparent amb text de victòria/derrota i estrelles obtingudes (⭐) al final de la partida.

### Colors del mapa

| Tipus      | Color hex  | Color visual    |
|------------|------------|-----------------|
| `pla`      | `0x90EE90` | verd clar       |
| `bosc`     | `0x228B22` | verd fosc       |
| `obstacle` | `0x808080` | gris            |
| `aigua`    | `0x4169E1` | blau            |
| `rail`     | `0x8B4513` | marró           |
| `inici`    | `0xFFD700` | groc/daurat     |
| `meta`     | `0xFF4500` | taronja vermell |

### Què falta / propers passos típics

- **Múltiples nivells:** ara `src/config/nivells.js` conté un sol nivell de prova.
- **Assets:** `public/icons.svg` existeix però no es carrega encara des de cap escena.
- **Tests:** no hi ha suite de proves automatitzades.
- **README:** actualment només conté el títol del projecte.

---
