# « Dans mon volcan » 🌋 — spec de design

**Date :** 2026-07-31
**Projet :** explorateur du volcan pour enfants (dès 4 ans), en français.
**Référence de style :** `../raphael_corps_humain` (« Dans mon corps »), dont on reprend
l'esprit, l'esthétique et les mécanismes d'interaction.

## Objectif

Une page web unique (`index.html`), sans dépendance ni installation, qui fait découvrir
à un enfant qui ne lit pas encore ce qu'est un volcan : à quoi il ressemble dehors,
ce qu'il cache dedans, d'où vient le magma, et comment se déroule une éruption.
Tout est cliquable et **lu à voix haute en français** (synthèse vocale du navigateur).

## Le mécanisme central : deux curseurs croisés

Contrairement au corps (un seul curseur de couches), le volcan a **deux axes** :

### Curseur COUPE (axe vertical de lecture, flèches ↑↓, 3 pastilles illustrées)

Trois vues en **fondu continu** (comme les 5 couches du corps) :

1. **Dehors** — le volcan dans son paysage : cône, forêt, village au pied, rivière,
   fumerolles au sommet, oiseaux, nuages.
2. **Dedans** — la grande coupe du strato-volcan : cratère, cheminée principale,
   cheminée secondaire, chambre magmatique, strates de laves et de cendres anciennes.
3. **Les profondeurs** — sous le volcan : croûte terrestre, manteau, deux plaques
   tectoniques qui se rencontrent (subduction simplifiée), zone de fusion où naît
   le magma, remontée vers la chambre.

### Curseur TEMPS (l'histoire de l'éruption, flèches ←→, 5 moments)

Cinq moments en **fondu continu** :

1. **Endormi** — tout est calme, un filet de fumée, les animaux vaquent.
2. **Le réveil** — le magma monte dans la cheminée, la terre tremble (léger shake
   visuel), grondements, les animaux s'enfuient.
3. **L'éruption !** — panache de cendres, bombes volcaniques, fontaines de lave,
   ciel qui rougeoie.
4. **Les coulées** — rivières de lave orange sur les flancs, le panache retombe,
   cendres qui tombent.
5. **Le repos** — la lave refroidit et devient roche noire, la fumée s'éteint,
   la vie revient (pousses vertes).

### Croisement des deux axes

Chaque vue réagit au temps. Exemples de combinaisons remarquables :

| | Endormi | Réveil | Éruption | Coulées | Repos |
|---|---|---|---|---|---|
| **Dehors** | filet de fumée | animaux qui fuient, tremblement | panache + bombes + fontaines | rivières de lave vers le village | flancs noircis, pousses vertes |
| **Dedans** | magma bas dans la chambre | magma qui monte dans la cheminée | cheminée pleine, jaillissement au cratère | niveau qui redescend | chambre calmée, bouchon qui se forme |
| **Profondeurs** | plaques immobiles, gouttes de magma | magma aspiré vers le haut | flux fort croûte → chambre | flux qui ralentit | retour au calme |

Il n'y a **pas 15 dessins séparés** : chaque vue est un ensemble de groupes SVG dont
les états (opacité, transformations, niveaux de magma) sont interpolés selon la
position du curseur temps.

## La science prise au sérieux

Comme l'anatomie de « Dans mon corps », la volcanologie n'est pas symbolique :

- Distinction **magma / lave** énoncée explicitement (le magma prend le nom de lave
  en sortant du volcan).
- Strato-volcan avec **strates** alternées (laves / cendres) visibles dans la coupe.
- **Chambre magmatique** à bonne profondeur relative, cheminée principale +
  cheminée secondaire avec petit cône adventif.
- **Panache** de type plinien, **bombes volcaniques** en fuseau, **fumerolles**.
- **Croûte / manteau / plaques** : subduction simplifiée mais orientée correctement
  (une plaque plonge sous l'autre, la fusion se produit au-dessus de la plaque
  plongeante).
- Vocabulaire exact : cratère, cheminée, cône, chambre magmatique, strate, panache,
  bombe volcanique, fumerolle, coulée, croûte, manteau, plaque tectonique, séisme…

**35 à 40 structures nommées et cliquables**, chacune avec : étiquette, explication
de deux phrases adaptée à un enfant de 4-5 ans, lecture à voix haute.

## Interactions (reprises de « Dans mon corps »)

- **Zones cliquables** : étiquette + explication + TTS français.
- **Loupe** déplaçable au doigt/souris pour regarder de près.
- **Bouton « Montrer les zones »** qui fait pulser toutes les zones cliquables
  (indispensable sur tablette, pas de survol).
- **Mini-jeu « Trouve la bonne partie ! »** : 3 questions, confettis et étoiles.
- **Clavier** : ↑↓ pour la coupe, ←→ pour le temps.
- Animations d'ambiance : bouillonnement de la chambre, fumerolles, scintillement
  de la lave, tremblement pendant le réveil, cendres qui tombent, oiseaux.
- `prefers-reduced-motion` coupe toutes les animations.

## Esthétique

- Même famille que « Dans mon corps » : fond planétarium nocturne, cartes en
  verre dépoli (`backdrop-filter`), gros titres Fredoka, UI Baloo 2, repli sur
  polices système (hors ligne OK).
- Palette **réchauffée** : lueurs orange/rouge magma montant du bas de l'écran,
  particules de cendres/braises flottantes à la place des bulles.
- Couleurs des 3 vues (pastilles, accents) : Dehors = vert/bleu paysage,
  Dedans = orange magma, Profondeurs = rouge sombre/pourpre.

## Technique

- **Un seul fichier** `index.html` : HTML, CSS et JS en ligne.
- SVG dessiné à la main, `viewBox` **paysage** (le volcan est plus large que haut,
  contrairement au corps), ~600-800 éléments.
- Aucune bibliothèque, aucune image bitmap, aucun appel réseau obligatoire
  (Google Fonts avec repli système).
- Les deux curseurs pilotent des interpolations d'opacité/transformation par
  groupe SVG ; les éléments communs (cône, ciel, sol) sont partagés entre vues.
- Synthèse vocale : `speechSynthesis` du navigateur, voix française.
  **Coupée par défaut au chargement** : un bouton son (visible et gros) permet de
  l'activer ; tant qu'elle est coupée, les étiquettes et explications s'affichent
  sans être lues.

## Hors périmètre (YAGNI)

- Pas de choix de type de volcan (rouge effusif / gris explosif) : **un seul
  volcan** générique très détaillé, de type strato-volcan.
- Pas de personnalisation du décor (jour/nuit, neige…).
- Pas de son autre que la synthèse vocale (pas de grondements audio).
- Pas de sauvegarde d'état, pas de backend.

## Critères de réussite

- La page s'ouvre en local (`open index.html`) et fonctionne hors ligne.
- Un enfant de 4-5 ans peut tout piloter seul : gros boutons, pastilles, curseurs.
- Chaque zone cliquée est nommée et expliquée à voix haute en français.
- Les deux curseurs se combinent sans état incohérent ni élément orphelin.
- Le mini-jeu fonctionne et félicite l'enfant.
- `prefers-reduced-motion` respecté ; utilisable au clavier ; utilisable sur tablette.
