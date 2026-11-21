# CARTODLE — DOCUMENT DE REPRISE DU PROJET

*Dernière mise à jour : 21 novembre 2026.*

## 1. Présentation du projet

Je développe un petit jeu web personnel appelé **Cartodle**.

Le jeu est développé en HTML, CSS et JavaScript natifs, sans framework.

Il fonctionne dans un simple fichier HTML local :
- Le fichier est `cartodle.html`.
- Il est ouvert directement dans un navigateur, sur ordinateur comme sur téléphone.
- **Toutes les données sont embarquées dans le fichier** : aucune connexion Internet n'est nécessaire et rien n'est téléchargé.

L'objectif est de proposer un jeu de recherche autour des Pokémon, de leurs numéros de Pokédex, de leurs types et de leurs stades évolutifs, avec des filtres inspirés des types du Jeu de Cartes à Collectionner Pokémon (JCC).

---

## 2. Principe du jeu

Le jeu possède un identifiant de départ.

L'identifiant initial est le numéro national de Pokédex **511** (Feuillajou).

Cet identifiant évolue au cours de la partie.

Le joueur choisit une direction avec deux boutons flèches :
- Flèche gauche : Plus petit (sélectionnée par défaut).
- Flèche droite : Plus grand.

Il sélectionne un Pokémon dans la liste des résultats. Son numéro national devient le nouvel ID de référence.

Il valide ensuite son tour avec le bouton **Valider le tour** :
- Si la direction choisie est « Plus petit », la borne maximale devient le numéro de l'ID moins 1.
- Si la direction choisie est « Plus grand », la borne minimale devient le numéro de l'ID plus 1.
- S'il n'y a plus de Pokémon dans la direction choisie, un message l'indique et le tour n'est pas validé.

L'intervalle doit toujours se réduire et ne doit jamais s'agrandir.

Exemple :
- ID de départ : 511.
- Choix : Plus petit.
- Validation : l'intervalle devient 1 à 510.

### Sélection automatique à la validation

Après la validation, le jeu **sélectionne automatiquement le Pokémon du milieu** des résultats filtrés, exactement comme si le joueur avait cliqué sur sa ligne :
- Son numéro devient le nouvel ID.
- Sa ligne est mise en évidence et centrée dans le listing.
- Les résultats sont triés par numéro, puis par nom. Le Pokémon du milieu est celui d'index `Math.floor(longueur / 2)`.
- S'il n'y a aucun résultat, l'ID ne change pas.
- Cette sélection n'a lieu qu'à la validation (pas lors d'un changement de filtre, ni à la réinitialisation).

Il n'y a pas de champ de saisie manuelle pour l'identifiant. Celui-ci est uniquement modifié en cliquant sur un Pokémon dans les résultats, ou automatiquement à la validation.

Le joueur peut cliquer sur un Pokémon même lorsqu'il est affiché en gris en mode « Afficher les autres ». Dans ce cas, l'ID change, mais les filtres ne changent pas.

---

## 3. Numéros de Pokédex et formes alternatives

C'est un point essentiel du projet.

### Règle fondamentale

**Le numéro national de Pokédex identifie l'espèce, tandis que la forme détermine les types utilisés pour le filtrage.**

Une même espèce peut donc apparaître plusieurs fois dans les résultats, avec le même numéro national, mais avec des types différents.

Exemple :

Sabelette classique :
- Numéro : 027.
- Type dans les jeux : Sol.
- Type correspondant dans le JCC selon notre table : Combat.

Sabelette d'Alola :
- Numéro : 027.
- Types dans les jeux : Glace et Acier.
- Types correspondants dans le JCC : Eau et Métal.

Les deux entrées sont présentes dans Cartodle, car elles ne correspondent pas aux mêmes filtres.

En revanche :
- Cliquer sur Sabelette classique définit l'ID à 027.
- Cliquer sur Sabelette d'Alola définit également l'ID à 027.
- Aucune forme ne reçoit un numéro de Pokédex artificiel.

### Formes retenues (décision définitive)

**Seules les formes régionales qui changent les types sont incluses** : Alola, Galar, Hisui et Paldea.

- 1025 espèces + 57 formes régionales = **1082 lignes**.
- Les formes Méga, Primal, Gigamax, les formes de combat, de climat ou de masque (Morphéo, Motisma, Necrozma, Ogerpon, etc.) sont **volontairement exclues**.
- Les formes régionales ont le même numéro et le même stade que leur espèce.
- Noms affichés : « Sabelette d'Alola », « Miaouss de Galar », « Caninos de Hisui », « Axoloto de Paldea »…
- Cas particuliers : « Tauros de Paldea (race Combat / Flamme / Aqua) » et « Darumacho de Galar » (forme standard uniquement).
- Les mentions de forme inutiles dans les noms ont été supprimées : Cheniti, Cheniselle, Sancoki et Tritosor s'affichent sans parenthèse.

---

## 4. Correspondance entre les types des jeux et les types du JCC

Les Pokémon sont stockés avec leurs types des jeux vidéo.

Ces types sont ensuite convertis en types JCC pour appliquer les filtres.

Voici la correspondance décidée :

| Type JCC | Types des jeux correspondants |
|---|---|
| Plante | Plante, Insecte, Poison |
| Feu | Feu |
| Eau | Eau, Glace |
| Électrique | Électrique |
| Psy | Psy, Spectre, Poison, Fée |
| Combat | Combat, Roche, Sol |
| Incolore | Normal, Vol, Dragon |
| Obscurité | Ténèbres, Poison |
| Métal | Acier |
| Dragon | Dragon |
| Fée | Fée |

Les correspondances qui se chevauchent sont intentionnelles :
- Poison peut correspondre à Plante, Psy et Obscurité.
- Dragon peut correspondre à Incolore et Dragon.

Il faut conserver exactement ces correspondances, sauf si je demande explicitement de les modifier.

### Cas particulier d'Arceus

Arceus, numéro 493, doit toujours passer le filtre de type, quel que soit le ou les types JCC sélectionnés.

En revanche, il doit toujours respecter :
- Les bornes de numéros.
- Le filtre de stade évolutif.

---

## 5. Filtre des stades évolutifs

Trois stades sont utilisés :

1. Base.
2. Niveau 1.
3. Niveau 2.

Les trois sont sélectionnés par défaut.

### Règle de classification (validée et appliquée)

Le stade est la profondeur dans la chaîne d'évolution, avec ces règles :

- **Un Pokémon dont la pré-évolution a été introduite dans une génération plus récente que lui compte comme Base**, et cette pré-évolution n'est pas comptée comme un stade.
  - Pichu : Base ; Pikachu : Base ; Raichu : Niveau 1.
  - Mélo et Mélofée : Base ; Mélodelfe : Niveau 1.
  - Toudoudou et Rondoudou : Base ; Grodoudou : Niveau 1.
  - Rozbouton et Rosélia : Base ; Rosérade : Niveau 1.
  - Ronflex, M. Mime, Kicklee, Tygnon, Leveinard, etc. : Base.
- **Tous les Pokémon considérés comme « bébé » sont Base.**
- Quand le bébé est de la même génération que son évolution, il compte comme un vrai stade :
  - Togepi : Base ; Togetic : Niveau 1 ; **Togekiss : Niveau 2**.
  - Riolu : Base ; Lucario : Niveau 1.
  - Toxel : Base ; Salarsen : Niveau 1.
- **Exception décidée à la main : Kapoera est Base.**

Les stades sont figés dans les données embarquées. Cette règle ne s'applique pas dans le jeu : si un Pokémon est ajouté plus tard, son stade doit être renseigné à la main.

---

## 6. Filtres de types

Les types JCC sont tous sélectionnés par défaut.

L'utilisateur peut sélectionner plusieurs types simultanément.

Règles :
- Un Pokémon correspond si au moins un de ses types JCC convertis est sélectionné.
- Les formes régionales sont filtrées individuellement.
- Arceus bénéficie de son exception.
- Deux boutons permettent de tout cocher ou tout décocher.

### Présentation

Pas de cases à cocher classiques : des pastilles colorées.
- Le nom du type apparaît sur sa couleur.
- Un type sélectionné apparaît avec sa couleur normale et un liseré clair.
- Un type désélectionné apparaît atténué et grisé.
- Cliquer sur la pastille sélectionne ou désélectionne le type.

### Couleurs

- **Filtres (types JCC)** : couleurs des énergies du JCC. Plante vert, Feu rouge, Eau bleu, Électrique jaune, Psy violet, Combat brun, Incolore gris clair, Obscurité bleu sombre, Métal gris acier, Dragon or, Fée rose.
- **Listing (types des jeux)** : couleurs officielles des types des jeux (par exemple Glace turquoise, Sol sable, Spectre violet foncé).
- Dans le listing, les badges affichent les **types des jeux** ; les filtres utilisent les **types JCC**. C'est voulu.
- La couleur du texte s'adapte automatiquement à la luminosité du fond.

---

## 7. Filtres informatifs : année, PV et rareté

Trois filtres sont présents. **Ils sont uniquement informatifs et ne modifient pas les résultats du jeu.** Chacun a une valeur minimale et maximale, sous forme de listes déroulantes (`<select>`), avec un garde-fou : le maximum est toujours supérieur ou égal au minimum.

- **Année** : de 1999 à l'année en cours (calculée automatiquement). Par défaut : min 1999, max année en cours.
- **PV** : de 30 à 380 par pas de 10. Par défaut : min 30, max 380. *Bornes vérifiées dans la base de cartes du JCC : 30 PV est le minimum des cartes Pokémon, 380 PV le maximum actuel (Mega Venusaur ex). Le plafond pourra monter avec de nouvelles extensions.*
- **Rareté** : 7 niveaux simplifiés. Par défaut : min Commune, max Ultra rares.

### Les 7 raretés

| # | Rareté | Symbole | Couleur |
|---|---|---|---|
| 1 | Commune | ● | gris clair |
| 2 | Peu commune | ◆ | vert |
| 3 | Rare | ★ | bleu |
| 4 | Rare Holo | ★ | violet |
| 5 | Double rare | ★★ | violet |
| 6 | Double rare brillante | ★★ | jaune |
| 7 | Ultra rares | ★★★ | arc-en-ciel |

L'année, les PV et la rareté ne sont pas affichés à côté des Pokémon dans les résultats.

Le bouton **Réinitialiser** remet ces filtres à leurs valeurs par défaut.

---

## 8. Affichage des résultats

Listing compact, une ligne par Pokémon, avec dans cet ordre :

**# Numéro | Nom | Type(s) | Stade**

Exemple :

| N° | Nom | Type(s) | Stade |
|---|---|---|---|
| 172 | Pichu | Électrique | Base |
| 025 | Pikachu | Électrique | Base |
| 026 | Raichu | Électrique | Niveau 1 |
| 027 | Sabelette | Sol | Base |
| 027 | Sabelette d'Alola | Glace + Acier | Base |

### Contraintes visuelles

- Une seule ligne par Pokémon, colonnes alignées comme dans un tableau.
- Numéro dans une colonne étroite, nom flexible, types et stade dans des colonnes dédiées.
- Les deux types tiennent sur la même ligne, sous forme de badges colorés.
- Hauteur des lignes réduite pour afficher un maximum de Pokémon.
- Largeur du listing limitée à environ 720 px sur grand écran, avec adaptation aux petits écrans.

### Positionnement du Pokémon central

Après filtrage, les résultats sont triés par numéro national. Le Pokémon du milieu (`Math.floor(longueur / 2)`) est sélectionné automatiquement à la validation (voir section 2) et positionné au centre de la zone de résultats. La liste a son propre défilement vertical.

### Mode « Afficher les autres »

Un interrupteur (style « switch »), placé **tout en haut de la section du listing**, affiche aussi les Pokémon qui ne correspondent pas aux filtres :
- Les Pokémon correspondants apparaissent normalement.
- Les autres apparaissent en gris et atténués, et restent cliquables.
- Cliquer sur un Pokémon gris change l'ID sans modifier les filtres.
- Dans ce mode, le centrage automatique du listing n'est pas appliqué.

---

## 9. Organisation générale de l'interface

Thème **sombre** uniquement.

Le titre en haut de page est **Cartodle**, compact, suivi d'un sélecteur de vues **Jeu | Voisins** (voir section 12). Le bouton **Réinitialiser** est dans l'en-tête, en haut à droite ; il n'apparaît que sur la vue Jeu.

### Desktop

Deux colonnes, sans défilement de la page :
- **À gauche** (panneau d'environ 250 px, qui tient dans la hauteur de l'écran) : zone d'ID, types JCC, stades, année, PV, rareté, puis le bouton **Valider le tour** sous les filtres.
- **À droite** : l'interrupteur « Afficher les autres », puis le listing, qui occupe **toute la hauteur disponible**.

### Zone d'identifiant

- Pas de titre « ID ».
- Le numéro de l'ID est affiché en grand, **entre deux boutons flèches SVG larges** (gauche = Plus petit, droite = Plus grand), très proches du nombre.
- La plage en cours (« N° 1 – 1025 ») est affichée juste dessous.

### Mobile (écran de 760 px ou moins)

- Une barre **« Filtres »** en haut permet d'enrouler ou de dérouler la section des filtres. Elle affiche aussi l'ID et la plage en cours.
- La section des filtres contient la zone d'ID, tous les filtres et le bouton **Valider le tour**.
- **Quand elle est visible, elle occupe toute la hauteur de l'écran** et le listing est masqué.
- **Valider le tour referme la section** et affiche le listing, avec la ligne du milieu sélectionnée et centrée.
- Au chargement, la section des filtres est ouverte.
- Champs, listes, pastilles et boutons agrandis pour le tactile (polices de 16 px dans les champs pour éviter le zoom automatique d'iPhone).
- **Le zoom est désactivé** (double-tap, pincement) et la sélection de texte est bloquée sur les boutons et les lignes.

---

## 10. Données

**Toutes les données sont embarquées** dans `cartodle.html`, dans la constante `POKEDEX`. Format d'une ligne :
`[numéro national, nom affiché, types des jeux, stade]` (stade : 0 = Base, 1 = Niveau 1, 2 = Niveau 2).

Les données ont été générées une fois à partir de :
- Dataset principal : `steffanossa/pokemon_dataset` (numéros, noms français, types, stades de départ).
- CSV de PokeAPI : `pokemon.csv`, `pokemon_types.csv`, `types.csv`, `pokemon_species_names.csv` (formes alternatives et types) et `pokemon_species.csv` (chaînes d'évolution, bébés, générations).

Elles ont ensuite été nettoyées : formes régionales uniquement, doublons supprimés (Rattatac d'Alola), noms des formes corrigés, stades recalculés selon la règle de la section 5.

### Modifier ou ajouter des données

Le tableau est figé. Pour ajouter une forme ou corriger un nom ou un stade, il faut soit modifier la ligne à la main dans `POKEDEX`, soit régénérer les données.

Le chargement ne dépend plus d'aucun réseau : il n'y a plus de gestion d'erreur de téléchargement.

---

## 11. État actuel du projet

Fonctionnalités en place :
- Jeu complet : ID, direction, validation avec réduction de l'intervalle, sélection automatique du Pokémon du milieu, réinitialisation.
- Filtres de types JCC (pastilles colorées), stades, et filtres informatifs année / PV / rareté.
- Listing sur une ligne avec formes régionales, badges colorés des types des jeux.
- Interrupteur « Afficher les autres ».
- Thème sombre, interface desktop sans défilement de la page.
- Version mobile avec section de filtres enroulable.
- Données embarquées, stades corrigés.

À vérifier ou à faire :
1. Tester le jeu dans des cas réels, sur desktop et sur téléphone, et corriger les bugs remontés.
2. Vérifier la liste des raretés (les bornes PV 30 à 380 sont vérifiées, mais le plafond peut évoluer avec les nouvelles extensions).
3. Vérifier la qualité des noms français des formes régionales.
4. Finaliser le CSS et l'apparence générale, notamment sur mobile.
5. Les filtres année, PV et rareté pourront un jour influencer les résultats ; ce n'est pas le cas pour l'instant. À terme, choisir un groupe de raretés complet ou une rareté précise.

---

## 12. Vue Voisins (outil d'aide à la résolution)

Une seconde vue, dans le même fichier, aide à résoudre le puzzle *Les Voisins* (issu du petit outil `Voisins`, intégré à Cartodle). Elle s'ouvre avec l'onglet **Voisins** de l'en-tête, ou directement avec `#voisins` dans l'adresse.

### Règles du puzzle
- 9 cartes Pokémon à ranger dans une grille 3×3.
- Deux cartes voisines (côtés uniquement, pas de diagonales) doivent partager au moins un point commun : un **type** (types des jeux, 1 ou 2 par carte), la **génération** (1 à 9) ou la **lignée** d'évolution.
- Une disposition gagne si chaque carte a un point commun avec chacune de ses voisines.

### Fonctionnement
- Saisie de 9 cartes : nom, types, génération, lignée. **Taper un nom remplit automatiquement** types, génération et lignée à partir des données de Cartodle (1025 espèces + formes régionales). Chaque champ reste modifiable pour une carte absente des données.
- La colonne « Liens » donne le nombre d'autres cartes avec un point commun ; elle passe en rouge quand la carte est trop isolée (moins de 2 liens pour un coin, moins de 4 pour le centre).
- **Sélection des types par pastilles colorées** (couleurs officielles des types des jeux, 18 types, 2 maximum par carte) :
  - **Desktop** : deux emplacements « Type 1 » et « Type 2 ». Un clic ouvre une palette flottante des 18 types ; choisir un type la referme. Le × vide l'emplacement. Choisir un type déjà présent dans l'autre emplacement le retire de celui-ci. La palette s'ouvre vers le haut pour les dernières cartes. Elle se ferme en cliquant ailleurs ou avec Échap.
  - **Mobile** : les types choisis sont des pastilles avec un ×, suivies d'un bouton « + Type » (absent quand la carte a déjà 2 types). La palette s'ouvre sous la carte, sans les types déjà pris. Pastilles et boutons de 35 px de haut pour le tactile.
  - Le remplissage automatique par nom remplit aussi ces pastilles.
- **Résoudre** explore toutes les dispositions (instantané). La **solution** s'affiche dans une grille 3×3 (bloc « Solution », centré sur mobile), et chaque paire de voisines est listée avec son point commun. Une seule solution est affichée : il n'y a ni flèches de navigation ni compteur « 1/1 ». Si aucune disposition n'existe, un message l'indique.
- Les symétries sont toujours masquées : une seule disposition est gardée par groupe de 8 symétries du carré (rotations et miroirs), car seule compte la recherche d'une solution. Il n'y a plus d'interrupteur.
- **Exemple** remplit les 9 cartes de démonstration (Némélios, Mentali, Pyroli, Magmar, Têtarte, Malvalame, Feuforêve, Mesmérella, Abra) et résout : 1 solution.
- **Vider** remet tout à zéro. Les 9 cartes sont **vides à l'arrivée** sur l'application ; le bouton **Exemple** les remplit.

### Décisions
- **Génération** : déduite du numéro national de l'espèce. Une forme régionale prend la génération de l'espèce d'origine (Sabelette d'Alola = génération 1).
- **Lignée** : nom du premier Pokémon de la chaîne d'évolution (Mentali et Pyroli : Évoli ; Mesmérella : Scrutella). Table embarquée `LINEAGE`.
  - **Exception pour 15 familles à bébé** : la lignée porte le nom de l'évolution juste au-dessus du bébé, et non celui du bébé. Pichu, Pikachu et Raichu donnent tous « Pikachu » ; Magby, Magmar et Maganon donnent « Magmar ».
  - Familles concernées : Pikachu, Mélofée, Rondoudou, Lippoutou, Élektek, Magmar, Marill, Qulbutoké, Rosélia, Éoko, Simularbre, M. Mime, Leveinard, Ronflex, Démanta (39 espèces).
  - **Pas d'exception** pour Debugant (lignée « Debugant »), ni pour Togepi, Riolu et Toxizap, dont l'évolution est de la même génération. Phione et Manaphy ont chacun leur propre lignée.
  - Les formes régionales suivent la lignée de leur espèce (M. Mime de Galar : « M. Mime »).
- Les formes Méga, Gigamax, etc. ne sont pas gérées par le remplissage automatique (saisie manuelle possible).
- Types affichés avec les noms des jeux, dont **Électrik** (et non « Électrique », nom utilisé pour le JCC dans les filtres de Cartodle). La comparaison se fait par identifiant interne.
- Mobile : la saisie est dans une section « Cartes » repliable ; **Résoudre** la referme et affiche les solutions (même principe que les filtres de la vue Jeu).

---

## 13. Ma façon de travailler

Je souhaite avancer progressivement.

Ordre de priorité actuel :

1. Tester le jeu dans des cas réels et corriger les bugs remontés.
2. Peaufiner l'interface desktop et mobile.
3. Vérifier les données restantes (noms, formes, filtres informatifs).
4. Faire évoluer les règles du jeu si besoin.

Ne modifie pas plusieurs aspects du jeu sans raison : conserve les fonctionnalités et les règles déjà validées.

Quand je demande une modification :
- Comprendre d'abord ce que je souhaite.
- Ne pas supprimer des fonctionnalités existantes.
- En cas de doute sur une règle ou un choix, me poser la question avant de coder.

**Le but final : un jeu fonctionnel, fiable, compact, visuellement lisible et facile à utiliser, sans avoir besoin de faire défiler constamment l'écran.**
