# « Dans mon volcan » — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Une page `index.html` unique qui fait explorer un volcan à un enfant de 4-5 ans : 3 vues en fondu (dehors / dedans / profondeurs) croisées avec 5 moments d'éruption, ~36 zones cliquables expliquées à voix haute (voix coupée par défaut), loupe, mini-jeu.

**Architecture:** Un seul fichier HTML/CSS/JS, SVG paysage dessiné à la main (`viewBox="0 0 900 560"`). Trois groupes de vue (`#g-dehors`, `#g-dedans`, `#g-prof`) en fondu piloté par `setCut(c)` (c ∈ [0,2]) ; les états d'éruption sont interpolés par `setTime(t)` (t ∈ [0,4]) via des tables d'opacité/valeurs par phase (`lerpArr`). Données des zones dans un objet `INFO`, synthèse vocale `speechSynthesis` **coupée par défaut**.

**Tech Stack:** HTML/CSS/JS vanilla, SVG à la main, Google Fonts (Fredoka, Baloo 2) avec repli système. Aucune bibliothèque, aucune image bitmap.

**Spec de référence :** `docs/superpowers/specs/2026-07-31-volcan-design.md` — la lire avant de commencer.
**Modèle de style :** `../raphael_corps_humain/index.html` — même esthétique (fond planétarium, cartes verre dépoli, gros boutons), à consulter pour le ton visuel et les patterns (TTS, loupe, jeu).

## Global Constraints

- **Un seul fichier livrable** : `index.html` (HTML, CSS, JS en ligne). Pas de fichier séparé, pas de build.
- **Aucune dépendance** : pas de lib JS, pas d'image bitmap, pas d'appel réseau obligatoire (fonts Google avec repli `"Chalkboard SE","Comic Sans MS",system-ui`).
- **Langue : français**, ton pour un enfant de 4-5 ans qui ne lit pas (tutoiement, phrases courtes).
- **Voix coupée par défaut** : `let soundOn = false;` au chargement ; bouton `#sound` initial `aria-pressed="false"`, libellé `🔇 Son coupé`.
- **`prefers-reduced-motion: reduce`** coupe toutes les animations décoratives.
- **Pas de type-checker ni linter configuré** (projet HTML statique). La vérification de chaque tâche = ouvrir la page dans un navigateur (Playwright MCP : `browser_navigate` sur `file:///Users/willow/Sites/_Claude_output/raphael_volcan/index.html`, puis `browser_console_messages` **zéro erreur** et `browser_take_screenshot` pour contrôle visuel). À défaut de Playwright : `open index.html` et inspection manuelle.
- **Commit à la fin de chaque tâche**, messages en français, suffixés `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`.
- Palette (à déclarer dans `:root`) :
  ```css
  --night-1:#160b26; --night-2:#251043; --night-3:#341a55;
  --cream:#fff4e2; --cream-soft:#d8cdf0;
  --lave:#ff5a2a; --magma:#ff7a3c; --braise:#ffb15c; --cendre:#8b8296;
  --c-dehors:#7ed98a; --c-dedans:#ffb15c; --c-prof:#ff6f7d;
  --skyday:#8fd3ff; --foret:#3f9d5a; --roche:#6b5b73; --roche-sombre:#3d3247;
  ```
- Coordonnées SVG communes (vues Dehors et Dedans) : `viewBox="0 0 900 560"`, ligne du sol `y=340`, sommet du volcan `(450,110)`, base du cône de `x=180` à `x=720`. La vue Profondeurs redessine sa propre scène (volcan miniature en haut, croûte/manteau en dessous).

---

### Task 1 : Squelette de la page (layout, fond, commandes inertes)

**Files:**
- Modify: `index.html` (créer)

**Interfaces:**
- Produces: structure DOM que toutes les tâches suivantes remplissent —
  `header` ; `.wrap` en grille 2 colonnes (`<main class="scene card">` à gauche,
  `<aside class="panel">` à droite) ; dans `.scene` : `<svg id="volcan" viewBox="0 0 900 560">`
  puis le curseur temps `#time` (`input type=range min=0 max=400 value=0`) avec ses
  5 repères `.moment[data-time="0..4"]` (🌙 Endormi, 😮 Le réveil, 🌋 L'éruption !,
  🔥 Les coulées, 🌱 Le repos) ; dans `.panel` : les 3 pastilles `.pill[data-cut="0..2"]`
  (🏞️ Dehors, 🌋 Dedans, 🌍 Les profondeurs) + curseur vertical `#cut`
  (`input type=range min=0 max=200 value=0`), la carte info `#info` (`#infoEmoji`,
  `#infoTitle`, `#infoText`), et la barre de boutons : `#say` (🔊 Réécouter),
  `#sound` (🔇 Son coupé, `aria-pressed="false"`), `#zones` (✨ Montrer les zones),
  `#loupeBtn` (🔍 La loupe), `#jeu` (🎯 Le jeu).

- [ ] **Step 1 : Écrire le squelette complet**

`<head>` : charset, viewport, `<title>Dans mon volcan — petit explorateur du volcan</title>`, description, fonts Fredoka + Baloo 2 (mêmes balises que le corps humain). CSS : palette du Global Constraints, fond planétarium **réchauffé** (dégradés nuit violacée + une lueur `radial-gradient` orange `#ff5a2a22` montant du bas), étoiles scintillantes (`body::before`, copie du pattern du corps), braises flottantes : `.braises i` = petits ronds orange qui **montent** en oscillant (`@keyframes monte`), 9 `<i>` avec tailles/délais variés. Cartes `.card` verre dépoli, `h1` en dégradé crème→orange avec un 🌋 animé (petit `@keyframes` de secousse douce), tagline « Tourne les deux boutons : découvre le volcan et fais-le entrer en éruption ! ». Curseurs stylés très gros (pouce ≥ 34px). Le SVG contient pour l'instant un simple triangle gris + ciel en dégradé (placeholder remplacé en Task 2). En bas de page, un `<footer>` discret « Fait avec ❤️ pour Raphaël ». Media query `max-width:900px` → 1 colonne. `@media (prefers-reduced-motion:reduce){ *,*::before,*::after{animation:none !important;transition:none !important} }`.

- [ ] **Step 2 : Vérifier dans le navigateur**

Playwright : `browser_navigate` → `file:///Users/willow/Sites/_Claude_output/raphael_volcan/index.html`, `browser_console_messages` (attendu : aucune erreur), `browser_take_screenshot` (attendu : layout 2 colonnes, fond nuit chaud, braises, curseurs visibles).

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Squelette : layout, fond planétarium chaud, commandes"
```

---

### Task 2 : Vue « Dehors » (paysage complet, état endormi)

**Files:**
- Modify: `index.html` (le `<svg id="volcan">`)

**Interfaces:**
- Produces: `#g-ciel` (fond partagé Dehors/Dedans : dégradé de ciel, soleil doux, 3 nuages dont `#d-nuage`) ; `#g-dehors` contenant les zones cliquables `class="zone"` avec ces **ids exacts** : `d-cone`, `d-cratere`, `d-fumee`, `d-foret`, `d-village`, `d-riviere`, `d-prairie`, `d-chemin`, `d-oiseaux`, `d-nuage`, `d-ancienne-coulee`, `d-volcanologue` ; plus les éléments pilotés par le temps (ids sans classe zone) : `#d-panache-ext`, `#d-coulees-ext`, `#d-lueur-cratere`, `#d-coulees-noires`, `#d-pousses`.

- [ ] **Step 1 : Dessiner le paysage**

Remplacer le placeholder. Contenu (état « endormi ») : ciel dégradé `--skyday`→crème à l'horizon dans `#g-ciel` ; cône du volcan (`d-cone`) aux coordonnées communes, flancs texturés (quelques traits de ravines), sommet tronqué avec lèvre de cratère (`d-cratere`) ; filet de fumerolles (`d-fumee`, 2-3 volutes `path` semi-transparentes animées lentement) ; forêt de conifères en 2 plans (`d-foret`, triangles verts variés, ~15 arbres) ; village de 4-5 maisons à toits rouges + clocher (`d-village`) en bas à droite ; rivière sinueuse bleue (`d-riviere`) descendant vers le village ; prairie fleurie (`d-prairie`, 5-6 fleurs) ; chemin en lacets vers le sommet (`d-chemin`) ; 3 oiseaux en « accents » (`d-oiseaux`) ; ancienne coulée noire figée sur le flanc gauche (`d-ancienne-coulee`) ; petite volcanologue au chapeau + jumelles près du chemin (`d-volcanologue`, ~8 formes simples). Éléments d'éruption **présents mais opacité 0** : grand panache gris (`#d-panache-ext`), coulées orange sur les flancs (`#d-coulees-ext`), lueur rouge au cratère (`#d-lueur-cratere`), coulées refroidies noires (`#d-coulees-noires`), pousses vertes (`#d-pousses`). Chaque `.zone` reçoit `tabindex="0"` et `role="button"`.

- [ ] **Step 2 : Vérifier dans le navigateur**

Console propre + screenshot : paysage lisible, joli, volcan dominant, aucun élément d'éruption visible.

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Vue Dehors : paysage complet à l'état endormi"
```

---

### Task 3 : Vue « Dedans » (la grande coupe)

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: coordonnées communes de Task 2 (même silhouette de cône).
- Produces: `#g-dedans` avec zones `class="zone"` ids : `c-cratere`, `c-cheminee`, `c-cheminee2`, `c-cone-adventif`, `c-chambre`, `c-magma`, `c-strates-lave`, `c-strates-cendres`, `c-bouchon`, `c-flanc`, `c-panache`, `c-bombes`, `c-fontaine`, `c-coulee` ; éléments pilotés par le temps : `#c-magma-colonne` (le magma dans la cheminée, piloté par `transform: translateY`), `#c-magma-niveau` (niveau dans la chambre), `#c-bulles` (bulles dans la chambre).

- [ ] **Step 1 : Dessiner la coupe**

Même silhouette de cône, mais « tranchée » : l'intérieur montre des **strates alternées** lave sombre / cendres claires (`c-strates-lave`, `c-strates-cendres`, 5-6 bandes obliques suivant la pente) ; **cheminée principale** verticale du cratère (`c-cratere`, entonnoir) vers le bas (`c-cheminee`, conduit ~26px de large, parois nettes) ; **cheminée secondaire** oblique (`c-cheminee2`) débouchant sur un **petit cône adventif** (`c-cone-adventif`) sur le flanc droit ; sous le sol (y=340→520), la **chambre magmatique** (`c-chambre`, grosse poche arrondie) remplie de magma orange vif (`c-magma`) avec dégradé incandescent + 4-5 bulles (`#c-bulles`, animation douce de remontée) ; **bouchon** de pierre au sommet de la cheminée (`c-bouchon`, visible seulement à t≈0 et t≈4 — opacité pilotée en Task 5) ; colonne de magma dans la cheminée (`#c-magma-colonne`, rect orange qui se translate verticalement, position basse par défaut) ; niveau de magma de la chambre (`#c-magma-niveau`). Au-dessus du cratère, en opacité 0 : **panache** gris en chou-fleur (`c-panache`), **bombes volcaniques** en fuseau avec traînées (`c-bombes`, 4-5), **fontaine de lave** (`c-fontaine`), **coulée** sur le flanc gauche (`c-coulee`). Le flanc externe restant est `c-flanc`. Zones avec `tabindex="0"` `role="button"`.

- [ ] **Step 2 : Vérifier dans le navigateur**

Pour voir la vue pendant le dev, mettre temporairement `#g-dehors{opacity:.15}`. Console propre + screenshot : coupe lisible, chambre + cheminées + strates identifiables. **Retirer le style temporaire avant commit.**

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Vue Dedans : coupe du strato-volcan"
```

---

### Task 4 : Vue « Les profondeurs » (croûte, manteau, subduction)

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: `#g-prof` (scène autonome, propre bande de ciel) avec zones ids : `p-croute`, `p-manteau`, `p-plaque-oce`, `p-plaque-cont`, `p-subduction`, `p-fusion`, `p-remontee`, `p-seisme`, `p-ocean` ; élément piloté par le temps : `#p-flux` (gouttes de magma qui montent le long de `p-remontee`).

- [ ] **Step 1 : Dessiner les profondeurs**

Scène complète (recouvre tout le viewBox quand visible) : bande de ciel étroite en haut (y 0→70) avec le **volcan miniature** (reprise simplifiée de la silhouette, ~90px de large, posé à x≈430) et un petit océan bleu à gauche (`p-ocean`, y≈60-70, vagues) ; **croûte terrestre** (`p-croute`, bande rocheuse y 70→200, texture de traits) fendue en deux plaques : **plaque océanique** (`p-plaque-oce`, à gauche, plus sombre et fine) qui **plonge en diagonale** sous la **plaque continentale** (`p-plaque-cont`, à droite) — la langue plongeante est `p-subduction` (flèche descendante intégrée) ; **manteau** (`p-manteau`, y 200→560, dégradé rouge sombre → pourpre, ondulations lentes) ; au-dessus de la plaque plongeante, la **zone de fusion** (`p-fusion`, amas de gouttes orange) d'où part la **remontée** (`p-remontee`, colonne de gouttes orange en pointillé montant jusque sous le volcan miniature) ; **étoiles de séisme** (`p-seisme`, 2-3 petits éclats ⚡ le long de la subduction). `#p-flux` = 5-6 gouttes sur la remontée animées vers le haut (`@keyframes`), vitesse pilotée plus tard. Zones avec `tabindex="0"` `role="button"`.

- [ ] **Step 2 : Vérifier dans le navigateur**

Temporairement `#g-dehors,#g-dedans{opacity:0}` ; console propre + screenshot : subduction orientée correctement (l'océanique plonge sous la continentale, fusion AU-DESSUS de la plaque plongeante). Retirer le style temporaire.

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Vue Profondeurs : croûte, manteau, subduction, naissance du magma"
```

---

### Task 5 : Curseur COUPE — fondu entre les 3 vues

**Files:**
- Modify: `index.html` (`<script>` principal, à créer en fin de `<body>`)

**Interfaces:**
- Consumes: `#g-ciel`, `#g-dehors`, `#g-dedans`, `#g-prof`, `#cut`, `.pill[data-cut]`.
- Produces: `setCut(c)` (c float 0..2, clampé), variable globale `cut`, et `currentView()` → 0|1|2 (vue dominante, `Math.round(cut)`), utilisés par les tâches 6-10.

- [ ] **Step 1 : Implémenter le fondu**

```js
const $ = s => document.querySelector(s), $$ = s => [...document.querySelectorAll(s)];
const clamp = (v,a,b) => Math.max(a, Math.min(b, v));
let cut = 0;
const VUES = ["#g-dehors","#g-dedans","#g-prof"];
function setCut(c){
  cut = clamp(c, 0, 2);
  VUES.forEach((sel,i) => {
    const g = $(sel), op = clamp(1 - Math.abs(cut - i), 0, 1);
    g.style.opacity = op;
    g.style.pointerEvents = (Math.round(cut) === i) ? "auto" : "none";
  });
  $("#g-ciel").style.opacity = clamp(2 - cut, 0, 1);   // le ciel partagé s'efface vers les profondeurs
  $("#cut").value = Math.round(cut * 100);
  $$(".pill").forEach(p => p.classList.toggle("active", +p.dataset.cut === Math.round(cut)));
  document.body.dataset.vue = Math.round(cut);          // accent de couleur par vue via CSS
}
const currentView = () => Math.round(cut);
$("#cut").addEventListener("input", e => setCut(e.target.value / 100));
$$(".pill").forEach(p => p.addEventListener("click", () => glideCut(+p.dataset.cut)));
function glideCut(target){                              // glissé doux ~500ms
  const from = cut, t0 = performance.now();
  (function step(now){
    const k = clamp((now - t0) / 500, 0, 1), e = k*(2-k); // easeOut
    setCut(from + (target - from) * e);
    if (k < 1) requestAnimationFrame(step);
  })(t0);
}
addEventListener("keydown", e => {
  if (e.key === "ArrowDown"){ e.preventDefault(); glideCut(clamp(Math.round(cut)+1,0,2)); }
  if (e.key === "ArrowUp"){ e.preventDefault(); glideCut(clamp(Math.round(cut)-1,0,2)); }
});
setCut(0);
```

CSS : `.pill.active` = bordure + lueur de la couleur de vue ; `body[data-vue="0"]{--accent:var(--c-dehors)}` etc.

- [ ] **Step 2 : Vérifier dans le navigateur**

Playwright : `browser_evaluate` → `setCut(1)` puis screenshot (coupe visible), `setCut(2)` puis screenshot (profondeurs), flèches ↑↓ via `browser_press_key`. Console propre.

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Curseur coupe : fondu continu entre les 3 vues"
```

---

### Task 6 : Curseur TEMPS — l'éruption interpolée sur les 3 vues

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: ids d'éléments pilotés des tâches 2-4 ; `#time`, `.moment[data-time]`.
- Produces: `setTime(t)` (t float 0..4), variable globale `time`, `lerpArr(arr,t)`, `currentPhase()` → 0..4 (`Math.round(time)`). Phases : 0 Endormi, 1 Réveil, 2 Éruption, 3 Coulées, 4 Repos.

- [ ] **Step 1 : Implémenter l'interpolation**

```js
let time = 0;
function lerpArr(arr, t){            // interpolation linéaire par morceaux sur [0..4]
  const i = clamp(Math.floor(t), 0, arr.length - 2), f = t - i;
  return arr[i] + (arr[i+1] - arr[i]) * f;
}
// Tables : 5 valeurs = [endormi, réveil, éruption, coulées, repos]
const OPACITES = [
  ["#d-fumee",          [ .8,  .3,   0,   0,  .5]],
  ["#d-panache-ext",    [  0,  .2,   1,  .35,  0]],
  ["#d-lueur-cratere",  [  0,  .5,   1,  .6,   0]],
  ["#d-coulees-ext",    [  0,   0,  .9,   1,   0]],
  ["#d-coulees-noires", [  0,   0,   0,  .2,   1]],
  ["#d-pousses",        [  0,   0,   0,   0,   1]],
  ["#d-oiseaux",        [  1,  .4,   0,   0,  .8]],   // les oiseaux fuient
  ["#c-panache",        [  0,  .15,  1,  .35,  0]],
  ["#c-bombes",         [  0,   0,   1,  .25,  0]],
  ["#c-fontaine",       [  0,   0,   1,  .4,   0]],
  ["#c-coulee",         [  0,   0,  .9,   1,  .15]],
  ["#c-bouchon",        [  1,  .25,  0,   0,  .8]],
  ["#p-seisme",         [  0,   1,  .7,  .2,   0]],
];
const MAGMA_Y = [120, 45, 0, 30, 90];       // translateY de #c-magma-colonne (0 = cheminée pleine)
const CHAMBRE_Y = [0, 8, 26, 14, 4];        // #c-magma-niveau descend pendant l'éruption
const ROUGE = [0, .12, .75, .4, .05];       // lueur du ciel, appliquée via --rougeoiement
function setTime(t){
  time = clamp(t, 0, 4);
  OPACITES.forEach(([sel, arr]) => { const el = $(sel); if (el) el.style.opacity = lerpArr(arr, time); });
  $("#c-magma-colonne").style.transform = `translateY(${lerpArr(MAGMA_Y, time)}px)`;
  $("#c-magma-niveau").style.transform  = `translateY(${lerpArr(CHAMBRE_Y, time)}px)`;
  document.documentElement.style.setProperty("--rougeoiement", lerpArr(ROUGE, time));
  document.body.classList.toggle("tremble", time > .5 && time < 1.6);   // séisme du réveil
  $("#g-prof").classList.toggle("flux-fort", time > 1.4 && time < 2.8); // remontée accélérée
  $("#time").value = Math.round(time * 100);
  $$(".moment").forEach(m => m.classList.toggle("active", +m.dataset.time === Math.round(time)));
}
const currentPhase = () => Math.round(time);
$("#time").addEventListener("input", e => setTime(e.target.value / 100));
$$(".moment").forEach(m => m.addEventListener("click", () => glideTime(+m.dataset.time)));
function glideTime(target){ /* même mécanique que glideCut, 600ms */ }
addEventListener("keydown", e => {
  if (e.key === "ArrowRight"){ e.preventDefault(); glideTime(clamp(Math.round(time)+1,0,4)); }
  if (e.key === "ArrowLeft"){ e.preventDefault(); glideTime(clamp(Math.round(time)-1,0,4)); }
});
setTime(0);
```

CSS : `.tremble #volcan{animation:tremble .18s linear infinite}` + `@keyframes tremble{25%{transform:translate(1.5px,-1px)}75%{transform:translate(-1.5px,1px)}}` ; `--rougeoiement` teinte une couche `<rect>` rouge posée sur le ciel (`#g-ciel .rouge{opacity:var(--rougeoiement)}`) ; `.flux-fort #p-flux *{animation-duration:.9s}` (contre ~2.5s au repos). Écrire `glideTime` en entier (copie de `glideCut` avec `setTime`).

- [ ] **Step 2 : Vérifier dans le navigateur**

`browser_evaluate` : `setTime(2)` + screenshot sur chaque vue (`setCut(0/1/2)`) — panache dehors, cheminée pleine dedans, flux fort en profondeur ; `setTime(4)` → coulées noires + pousses. Console propre.

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Curseur temps : éruption interpolée sur les 3 vues"
```

---

### Task 7 : Zones cliquables + données INFO + « Montrer les zones »

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: toutes les `.zone` des tâches 2-4, `setCut`, carte `#info`.
- Produces: `INFO` (35 entrées), `showInfo(id)`, `speakText(id)` → string, bouton `#zones` fonctionnel. `say(text)` n'existe pas encore : Task 8 la fournit ; en attendant, déclarer `let say = () => {};` (remplacée en Task 8).

- [ ] **Step 1 : Écrire les données**

```js
const INFO = {
  // ---- Dehors
  "d-cone":{n:"Le volcan",em:"🌋",t:"Voici le volcan ! C'est une montagne un peu spéciale : elle s'est construite toute seule, couche après couche, avec la lave de ses éruptions."},
  "d-cratere":{n:"Le cratère",em:"🕳️",t:"Tout en haut, il y a un grand trou : le cratère. C'est la bouche du volcan, c'est par là que tout sort !"},
  "d-fumee":{n:"Les fumerolles",em:"💨",t:"Ces petites fumées s'appellent des fumerolles. Elles montrent que le volcan respire, même quand il dort."},
  "d-foret":{n:"La forêt",em:"🌲",t:"Autour du volcan, la forêt pousse très bien. Les cendres des vieilles éruptions rendent la terre très nourrissante pour les plantes."},
  "d-village":{n:"Le village",em:"🏡",t:"Des gens habitent tout près, parce que la terre du volcan fait pousser de beaux légumes. Pas de panique : on surveille le volcan jour et nuit."},
  "d-riviere":{n:"La rivière",em:"💧",t:"La pluie qui tombe sur le volcan devient une rivière fraîche. Elle descend la pente en chantant jusqu'au village."},
  "d-prairie":{n:"La prairie",em:"🌼",t:"Sur les vieilles laves, des fleurs ont poussé. La nature revient toujours, même après une éruption."},
  "d-chemin":{n:"Le chemin",em:"🥾",t:"Ce petit chemin monte en zigzag jusqu'au sommet. C'est par là que passent les savants pour aller voir le cratère."},
  "d-oiseaux":{n:"Les oiseaux",em:"🐦",t:"Les animaux sentent le volcan avant nous ! Quand la terre commence à trembler, les oiseaux s'envolent très loin."},
  "d-nuage":{n:"Les nuages",em:"☁️",t:"Ceux-là sont des vrais nuages de pluie, tout doux. Ne les confonds pas avec le panache du volcan, qui est plein de cendres !"},
  "d-ancienne-coulee":{n:"La vieille coulée",em:"🪨",t:"Cette grande langue noire, c'est de la lave d'une ancienne éruption. En refroidissant, elle est devenue de la roche toute dure."},
  "d-volcanologue":{n:"La volcanologue",em:"👩‍🔬",t:"C'est la savante des volcans ! Avec ses appareils, elle écoute le volcan pour deviner quand il va se réveiller et prévenir le village."},
  // ---- Dedans
  "c-cratere":{n:"Le cratère",em:"🕳️",t:"Le cratère, c'est l'entonnoir au sommet du volcan. Pendant l'éruption, le magma jaillit par ici et devient de la lave."},
  "c-cheminee":{n:"La cheminée",em:"🪈",t:"La cheminée est le grand tuyau du volcan. Elle part de la chambre magmatique et monte jusqu'au cratère : le magma grimpe par là."},
  "c-cheminee2":{n:"La petite cheminée",em:"🪄",t:"Parfois le magma se faufile par un tuyau de côté. On dit une cheminée secondaire : elle fait un petit volcan sur le flanc du grand."},
  "c-cone-adventif":{n:"Le petit cône",em:"⛰️",t:"Au bout de la petite cheminée, un mini-volcan a poussé sur le flanc du grand. C'est son petit frère !"},
  "c-chambre":{n:"La chambre magmatique",em:"🫙",t:"Sous le volcan se cache une énorme poche remplie de roche fondue toute chaude. C'est le réservoir du volcan : la chambre magmatique."},
  "c-magma":{n:"Le magma",em:"🍯",t:"Le magma, c'est de la roche si chaude qu'elle a fondu, comme du chocolat ! Quand il sort du volcan, il change de nom : on l'appelle la lave."},
  "c-strates-lave":{n:"Les couches de lave",em:"🥞",t:"Regarde les bandes sombres : chaque éruption a laissé une couche de lave durcie. Le volcan a grandi comme une pile de crêpes !"},
  "c-strates-cendres":{n:"Les couches de cendres",em:"🍰",t:"Entre les laves, il y a des couches plus claires : ce sont les cendres retombées après les grosses éruptions."},
  "c-bouchon":{n:"Le bouchon",em:"🧀",t:"Quand le volcan dort longtemps, la lave durcit dans la cheminée et fait un bouchon de pierre. Au réveil, le magma doit pousser très fort pour l'ouvrir !"},
  "c-flanc":{n:"Le flanc",em:"📐",t:"Le flanc, c'est la pente du volcan. Les coulées de lave glissent dessus, et les couches s'empilent pour le faire grandir."},
  "c-panache":{n:"Le panache",em:"🌫️",t:"Cet immense nuage gris est plein de cendres et de gaz brûlants. Il peut monter plus haut que les avions !"},
  "c-bombes":{n:"Les bombes volcaniques",em:"🏉",t:"Le volcan crache des morceaux de lave qui volent ! En tournant dans le ciel, ils prennent une forme de ballon de rugby."},
  "c-fontaine":{n:"La fontaine de lave",em:"⛲",t:"Au plus fort de l'éruption, la lave jaillit du cratère comme une fontaine de feu. Elle éclabousse tout autour !"},
  "c-coulee":{n:"La coulée de lave",em:"🔥",t:"La lave descend la pente comme une rivière orange, très lente et très très chaude. Sur son passage, tout brûle."},
  // ---- Profondeurs
  "p-croute":{n:"La croûte terrestre",em:"🥖",t:"La croûte, c'est la peau dure de la Terre : nous marchons dessus ! Comparée à toute la Terre, elle est fine comme la coquille d'un œuf."},
  "p-manteau":{n:"Le manteau",em:"🌡️",t:"Sous la croûte, la roche est si chaude qu'elle devient molle et bouge tout doucement, comme une pâte à gâteau qu'on remue."},
  "p-plaque-oce":{n:"La plaque de l'océan",em:"🌊",t:"La croûte est découpée en grands morceaux qui flottent : les plaques. Celle-ci porte l'océan sur son dos."},
  "p-plaque-cont":{n:"La plaque du continent",em:"🏔️",t:"Cette plaque-là porte la terre où nous vivons, avec ses montagnes et notre volcan."},
  "p-subduction":{n:"La plongée",em:"🛝",t:"Les deux plaques se poussent, et la plaque de l'océan glisse sous l'autre, comme sur un toboggan. Les savants appellent ça la subduction."},
  "p-fusion":{n:"La naissance du magma",em:"✨",t:"En s'enfonçant, la plaque chauffe tellement que la roche au-dessus se met à fondre. C'est ici que naît le magma !"},
  "p-remontee":{n:"La remontée",em:"🎈",t:"Le magma est plus léger que la roche autour de lui. Alors il monte, bulle après bulle, jusqu'à la chambre magmatique du volcan."},
  "p-seisme":{n:"Le tremblement de terre",em:"⚡",t:"Quand les plaques frottent l'une contre l'autre, la terre tremble ! Ces petits séismes annoncent parfois le réveil du volcan."},
  "p-ocean":{n:"L'océan",em:"🐳",t:"Tout là-bas, c'est la mer. C'est sous elle que commence le grand toboggan des plaques."},
};
```

- [ ] **Step 2 : Brancher les zones et la carte info**

```js
let say = () => {};                       // remplacée par la vraie TTS en Task 8
let currentId = null;
function speakText(id){ const z = INFO[id]; return `${z.n}. ${z.t}`; }
function showInfo(id){
  const z = INFO[id]; if (!z) return;
  currentId = id;
  $("#infoEmoji").textContent = z.em;
  $("#infoTitle").textContent = z.n;
  $("#infoText").textContent  = z.t;
  $("#info").classList.add("open");
  $$(".zone.selected").forEach(el => el.classList.remove("selected"));
  document.getElementById(id)?.classList.add("selected");
  say(speakText(id));
}
$$(".zone").forEach(z => {
  z.addEventListener("click", () => showInfo(z.id));
  z.addEventListener("keydown", e => { if (e.key === "Enter" || e.key === " "){ e.preventDefault(); showInfo(z.id); } });
});
$("#zones").addEventListener("click", () => {
  document.body.classList.add("show-zones");
  setTimeout(() => document.body.classList.remove("show-zones"), 3500);
});
```

CSS : `.zone{cursor:pointer}` ; `.zone.selected` = contour crème lumineux (`filter:drop-shadow`) ; `.show-zones .zone` = `@keyframes pulse-zone` (halo qui pulse, 3 pulsations) ; carte `#info` avec transition d'apparition, gros emoji, titre Fredoka couleur `--accent`. Vérifier que **chaque id de `INFO` existe dans le SVG** (boucle console : `Object.keys(INFO).filter(id=>!document.getElementById(id))` doit renvoyer `[]`).

- [ ] **Step 3 : Vérifier dans le navigateur**

Screenshot après clic sur le volcan (étiquette + carte remplie), test `#zones` (pulsation), test clavier (Tab + Entrée sur une zone). Console : la boucle de cohérence renvoie `[]`, zéro erreur.

- [ ] **Step 4 : Commit**

```bash
git add index.html && git commit -m "36 zones cliquables, carte info, montrer les zones"
```

---

### Task 8 : Synthèse vocale (coupée par défaut) + boutons son

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `speakText(id)`, `currentId`, boutons `#say`, `#sound`.
- Produces: vraie fonction `say(text)` (remplace le stub), `soundOn` (**false par défaut**).

- [ ] **Step 1 : Implémenter la voix**

Reprendre le pattern du corps humain (`../raphael_corps_humain/index.html`, lignes ~2458-2480 et ~2644-2650) :

```js
let soundOn = false;                                  // COUPÉ PAR DÉFAUT (spec)
let voice = null;
function pickVoice(){
  if (!("speechSynthesis" in window)) return;
  const vs = speechSynthesis.getVoices();
  voice = vs.find(v => /fr[-_]FR/i.test(v.lang) && /audrey|thomas|amelie|amélie|marie/i.test(v.name))
       || vs.find(v => /^fr/i.test(v.lang)) || null;
}
if ("speechSynthesis" in window){ pickVoice(); speechSynthesis.onvoiceschanged = pickVoice; }
say = (text) => {                                     // remplace le stub de Task 7
  if (!soundOn || !("speechSynthesis" in window) || !text) return;
  try{
    speechSynthesis.cancel();
    const u = new SpeechSynthesisUtterance(text);
    u.lang = "fr-FR"; u.rate = .92; u.pitch = 1.05;
    if (voice) u.voice = voice;
    speechSynthesis.speak(u);
  }catch(e){ /* pas de voix : on continue sans son */ }
};
$("#sound").addEventListener("click", () => {
  soundOn = !soundOn;
  $("#sound").setAttribute("aria-pressed", String(soundOn));
  $("#sound").textContent = soundOn ? "🎧 Son allumé" : "🔇 Son coupé";
  if (!soundOn && "speechSynthesis" in window) speechSynthesis.cancel();
  else say("Le son est allumé !");
});
$("#say").addEventListener("click", () => { if (currentId) say(speakText(currentId)); });
```

- [ ] **Step 2 : Vérifier dans le navigateur**

Au chargement : bouton `🔇 Son coupé`, `aria-pressed="false"`, aucun son au clic sur une zone (la carte s'affiche quand même). Après activation : le clic parle. Console propre.

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Synthèse vocale française, coupée par défaut"
```

---

### Task 9 : La loupe

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `#volcan` (le SVG), `#loupeBtn`.
- Produces: mode loupe activable/désactivable, `loupeOn` (bool).

- [ ] **Step 1 : Implémenter la loupe**

Technique : donner au contenu du SVG un id englobant `#scene` (`<g id="scene">` autour de tout), puis une **2e balise `<svg id="lens">`** absolue (140×140, ronde : `border-radius:50%`, bordure crème épaisse + petit manche décoratif) contenant `<use href="#scene">`. Le zoom se fait en réglant le `viewBox` de `#lens` sur une fenêtre de 900/2.2 ≈ 409×?? centrée sur le point survolé (conversion écran→viewBox avec `getBoundingClientRect`). La loupe suit `pointermove` sur la scène (souris **et** doigt, via `pointer` events + `touch-action:none` quand active). `#loupeBtn` bascule `loupeOn`, classe `lens-on` sur `.scene`, et le bouton devient actif visuellement. Important : dans `#lens`, forcer l'opacité des groupes à refléter l'état courant — comme `<use>` clone le DOM réel, les styles inline suivent automatiquement ; vérifier que c'est bien le cas.

```js
let loupeOn = false;
const lens = $("#lens"), sceneSvg = $("#volcan"), Z = 2.2;
$("#loupeBtn").addEventListener("click", () => {
  loupeOn = !loupeOn;
  $(".scene").classList.toggle("lens-on", loupeOn);
  $("#loupeBtn").classList.toggle("active", loupeOn);
  lens.style.display = loupeOn ? "block" : "none";
});
sceneSvg.addEventListener("pointermove", e => {
  if (!loupeOn) return;
  const r = sceneSvg.getBoundingClientRect();
  const vx = (e.clientX - r.left) / r.width * 900, vy = (e.clientY - r.top) / r.height * 560;
  const w = 900 / Z, h = 560 / Z;
  lens.setAttribute("viewBox", `${clamp(vx - w/2, 0, 900 - w)} ${clamp(vy - h/2, 0, 560 - h)} ${w} ${h}`);
  lens.style.left = (e.clientX - r.left - 70) + "px";
  lens.style.top  = (e.clientY - r.top  - 70) + "px";
});
```

- [ ] **Step 2 : Vérifier dans le navigateur**

Activer la loupe, `browser_evaluate` d'un `pointermove` simulé ou survol réel + screenshot : cercle grossissant qui suit, contenu agrandi cohérent avec la vue et le temps courants. Console propre.

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Loupe grossissante déplaçable"
```

---

### Task 10 : Mini-jeu « Trouve la bonne partie ! »

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `INFO`, `showInfo`, `say`, `glideCut`, `currentView`, `.zone`.
- Produces: `startQuiz()` branchée sur `#jeu`, overlay `#quiz` (bandeau question + score étoiles), confettis.

- [ ] **Step 1 : Implémenter le jeu**

```js
const QUIZ_POOL = [
  {q:"Trouve la bouche du volcan : le cratère !", ids:["d-cratere","c-cratere"]},
  {q:"Où le magma attend-il, bien au chaud, sous le volcan ?", ids:["c-chambre","c-magma"]},
  {q:"Trouve le grand tuyau où grimpe le magma : la cheminée !", ids:["c-cheminee"]},
  {q:"Qui écoute le volcan pour prévenir le village ?", ids:["d-volcanologue"]},
  {q:"Trouve l'endroit où naît le magma, tout au fond !", ids:["p-fusion"]},
  {q:"Montre la peau dure de la Terre : la croûte !", ids:["p-croute"]},
  {q:"Trouve la rivière de feu qui descend la pente !", ids:["c-coulee","d-coulees-ext"].filter(id=>INFO[id]),},
  {q:"Où habitent les gens, tout près du volcan ?", ids:["d-village"]},
];
let quiz = null;   // {questions:[...3], index, stars}
function startQuiz(){
  const qs = [...QUIZ_POOL].sort(() => Math.random() - .5).slice(0, 3);
  quiz = { questions: qs, index: 0, stars: 0 };
  $("#quiz").classList.add("open");
  askNext();
}
function askNext(){
  if (quiz.index >= 3){ endQuiz(); return; }
  const q = quiz.questions[quiz.index];
  const targetVue = +document.getElementById(q.ids[0]).closest("[id^=g-]").id.replace(/g-(dehors|dedans|prof)/, m => ({ "g-dehors":0, "g-dedans":1, "g-prof":2 }[m]));
  glideCut(targetVue);                       // on amène l'enfant sur la bonne vue
  $("#quizQ").textContent = q.q;
  say(q.q);
}
function onZoneClickInQuiz(id){
  const q = quiz.questions[quiz.index];
  if (q.ids.includes(id)){
    quiz.stars++; quiz.index++;
    confetti(); say("Bravo !"); renderStars();
    setTimeout(askNext, 1200);
  } else {
    say("Presque ! Cherche encore.");
    $("#quizQ").classList.add("shake"); setTimeout(() => $("#quizQ").classList.remove("shake"), 500);
  }
}
function endQuiz(){
  $("#quizQ").textContent = "Bravo, champion des volcans ! ⭐⭐⭐";
  say("Bravo, tu es un vrai petit volcanologue !");
  confetti(); confetti();
  setTimeout(() => { $("#quiz").classList.remove("open"); quiz = null; }, 3000);
}
```

Brancher dans le handler de clic des zones (Task 7) : `if (quiz) { onZoneClickInQuiz(z.id); return; } showInfo(z.id);`. **Pendant le jeu, activer les pointer-events de toutes les vues** pour que l'enfant puisse se tromper de vue sans blocage — ou plus simple : les questions ciblent la vue affichée par `glideCut`, et les deux curseurs restent utilisables. Confettis : ~24 `<i>` emoji/carrés colorés positionnés en absolu qui tombent avec rotations (`@keyframes chute`), supprimés après 2,5 s (pattern du corps humain). `renderStars()` : remplit `#quizStars` avec `"⭐".repeat(quiz.stars)`. `#quiz` = bandeau fixé en haut de la scène, gros texte, bouton ✖️ pour quitter. Simplifier `QUIZ_POOL[6]` : garder uniquement `ids:["c-coulee"]` (l'id `d-coulees-ext` n'est pas une zone).

- [ ] **Step 2 : Vérifier dans le navigateur**

Lancer le jeu, répondre juste (confettis + étoile), répondre faux (message doux), finir les 3 questions (félicitations). Console propre.

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Mini-jeu Trouve la bonne partie, confettis et étoiles"
```

---

### Task 11 : Ambiance, accessibilité, polish final

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: tout ce qui précède.

- [ ] **Step 1 : Passe d'ambiance et d'accessibilité**

- Animations douces permanentes : bulles du magma (`#c-bulles`), volutes de fumerolles, oiseaux qui planent, ondulation du manteau, scintillement des coulées actives (petit `filter` ou opacité oscillante), gouttes de `#p-flux`.
- Vérifier `prefers-reduced-motion` : tout s'arrête, la page reste 100 % utilisable.
- Accessibilité : `aria-label` français sur chaque `.zone` (depuis `INFO[id].n`, boucle JS au chargement), `aria-live="polite"` sur `#infoText`, labels sur les curseurs (`aria-label="Choisir la vue"` / `"Faire avancer l'éruption"`), focus visibles.
- Responsive : tester 390px de large (colonne unique, SVG pleine largeur, curseur temps utilisable au pouce).
- Chasse aux détails : z-index de la loupe vs bandeau quiz, le `tremble` ne casse pas la loupe, aucun élément orphelin dans les combinaisons extrêmes (`setCut(2)`+`setTime(2)`, etc. — passer les 15 combinaisons `browser_evaluate` + screenshots).

- [ ] **Step 2 : Vérification complète**

Playwright : les 15 combinaisons vue×phase en screenshots, console vierge sur tout le parcours (chargement → clics → jeu → loupe). Test clavier complet (↑↓←→, Tab, Entrée).

- [ ] **Step 3 : Commit**

```bash
git add index.html && git commit -m "Ambiance, accessibilité, responsive, polish"
```

---

### Task 12 : README + publication GitHub Pages

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: le produit fini.

- [ ] **Step 1 : Écrire le README**

Sur le modèle de `../raphael_corps_humain/README.md` : titre « Dans mon volcan 🌋 », lien Pages `https://w3cdotorg.github.io/dans-le-volcan/`, description (2 curseurs croisés, 3 vues × 5 moments, ~36 structures nommées, voix FR coupée par défaut), tableau « La volcanologie est prise au sérieux » (une ligne par vue : ce qu'elle contient exactement), utilisation (`open index.html`, serveur local pour tablette), section technique (un seul fichier, SVG à la main, aucune dépendance), licence MIT.

- [ ] **Step 2 : Activer GitHub Pages et vérifier**

```bash
git add README.md && git commit -m "README" && git push
gh api repos/w3cdotorg/dans-le-volcan/pages -X POST -f "source[branch]=main" -f "source[path]=/" 2>/dev/null || true
```

Attendre le déploiement (`gh api repos/w3cdotorg/dans-le-volcan/pages --jq .status`), puis ouvrir l'URL publique et refaire un tour rapide (console + une interaction de chaque type).

- [ ] **Step 3 : Commit final / push**

```bash
git push
```

---

## Self-review (fait à l'écriture du plan)

- **Couverture de la spec :** 2 curseurs croisés (T5, T6), 3 vues (T2-T4), 5 moments interpolés (T6), 35 zones + TTS off par défaut (T7, T8), loupe (T9), jeu (T10), reduced-motion + tablette + clavier (T1, T11), un seul fichier sans dépendance (contrainte globale), README/Pages (T12). Distinction magma/lave dans `INFO["c-magma"]`, subduction orientée (T4). ✔
- **Placeholders :** aucun TBD ; `glideTime` explicitement « copie de glideCut avec setTime » avec consigne de l'écrire en entier. ✔
- **Cohérence des noms :** ids SVG des tâches 2-4 = clés de `INFO` (T7) = cibles de `OPACITES` (T6, ids pilotés non-zones exclus d'INFO) = `QUIZ_POOL` (T10, avec correction de la ligne 7 notée). `say` stub (T7) remplacé (T8). ✔
