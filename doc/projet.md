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

Le titre en haut de page est simplement **Cartodle**, compact. Le bouton **Réinitialiser** est dans l'en-tête, en haut à droite.

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

## 12. Ma façon de travailler

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
