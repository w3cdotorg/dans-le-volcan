# Dans mon volcan 🌋

**→ [Ouvrir le site](https://w3cdotorg.github.io/dans-le-volcan/)**

Un petit explorateur du volcan pour les enfants (dès 4 ans), en français.
Une page web unique, sans dépendance ni installation : on ouvre `index.html` et c'est parti.

Petit frère de [*Dans mon corps*](https://w3cdotorg.github.io/dans-mon-corps/) : même
esthétique de planétarium et de cartes en verre dépoli, réchauffée cette fois par la lueur
du magma et des braises flottantes.

## Ce que ça fait

- **Deux curseurs croisés.** Le premier fait fondre la scène de **Dehors** (le paysage autour
  du volcan) vers **Dedans** (coupe du volcan) puis vers **les Profondeurs** (la Terre sous le
  volcan). Le second fait avancer une éruption en cinq moments : **Endormi 🌙 → Réveil 😮 →
  Éruption 🌋 → Coulées 🔥 → Repos 🌱**. Les deux sont indépendants — on peut regarder n'importe
  lequel des cinq moments depuis n'importe laquelle des trois vues (3 × 5 combinaisons) — et
  chacun se pilote au doigt/à la souris (glissière continue), par pastille, ou au clavier.
- **35 zones cliquables** (12 dehors, 14 dans la coupe du volcan, 9 dans les profondeurs) :
  chacune affiche un nom et une explication de deux phrases, et **tout est lu à voix haute en
  français** (synthèse vocale du navigateur) — mais **la voix est coupée par défaut** ; un
  bouton 🔇 Son coupé / 🎧 Son allumé l'active.
- **Une loupe** qui suit le doigt ou la souris, pour aller regarder la chambre magmatique ou
  une bombe volcanique de près.
- **Un mini-jeu** « Trouve la bonne partie ! » : trois questions tirées d'une banque de huit,
  avec étoiles et confettis à la fin.
- **Un bouton « Montrer les zones »** qui fait pulser tous les endroits où l'on peut appuyer
  (indispensable sur tablette, où il n'y a pas de survol).
- Animations d'ambiance : fumerolles, bulles dans la chambre magmatique, oiseaux qui
  s'envolent avant un séisme — toutes coupées si `prefers-reduced-motion` est actif.

## La volcanologie est prise au sérieux

Les trois vues ne sont pas des schémas symboliques :

| Vue | Contenu |
|---|---|
| **Dehors** | Le cône, le cratère, les fumerolles (le volcan « respire » même endormi), le chemin d'accès des volcanologues, la forêt et le village qui prospèrent sur les terres fertilisées par les cendres, une ancienne coulée refroidie en roche, une prairie qui a repoussé dessus, les oiseaux qui sentent un séisme avant nous, et une volcanologue qui surveille l'activité |
| **Dedans** | Chambre magmatique, cheminée principale **et cheminée secondaire** alimentant un **cône adventif** (un vrai petit volcan satellite sur le flanc), distinction explicite **magma** (sous terre) / **lave** (le même matériau, une fois sorti), strates de lave **et** strates de cendres empilées éruption après éruption, un **bouchon** de lave durcie qui doit être percé au réveil, panache de cendres, bombes volcaniques **en fuseau**, fontaine de lave, coulée |
| **Les profondeurs** | **Subduction** orientée d'une plaque océanique sous une plaque continentale, manteau, la **fusion** qui a lieu dans la roche *au-dessus* de la plaque plongeante (pas dans la plaque elle-même — le raccourci qu'on lit trop souvent), la remontée du magma par flottabilité jusqu'à la chambre, les séismes précurseurs, et une croûte terrestre fine « comme une coquille d'œuf » |

## Utilisation

```bash
open index.html          # macOS
```

Pour y accéder depuis une tablette ou un autre poste du réseau local :

```bash
python3 -m http.server 8000 --bind 0.0.0.0
# puis http://<ip-de-la-machine>:8000/index.html
```

## Technique

Un seul fichier : HTML, CSS et JavaScript en ligne, dessin en SVG écrit à la main. Aucune
bibliothèque, aucune image bitmap, aucun appel réseau au runtime hormis les deux polices
(Fredoka, Baloo 2) chargées depuis Google Fonts avec repli sur des polices système, donc la
page reste correcte hors ligne. `prefers-reduced-motion` coupe les animations, toutes les
zones sont accessibles au clavier avec `aria-label` tenu à jour depuis la même source que le
texte affiché, et la mise en page s'adapte à la tablette comme au téléphone.

## Licence

MIT.
