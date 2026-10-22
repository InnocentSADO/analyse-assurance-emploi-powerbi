# Dictionnaire des données

## 1. Format des fichiers sources (Statistique Canada)

| Caractéristique | Valeur | Traitement dans Power Query |
|---|---|---|
| Encodage | UTF-8 | Origine du fichier : 65001 |
| Séparateur | Point-virgule (;) | Délimiteur précisé à l'importation |
| Décimales | Point (ex. « 6.8 ») | Conversion avec les paramètres régionaux Anglais (Canada) |
| Période | Texte « AAAA-MM » | Conversion en date (1er du mois), indépendante des paramètres régionaux de Windows |
| Facteur scalaire | « unités » | Aucun multiplicateur : les valeurs sont des personnes réelles |

## 2. Tables de faits

### Fait_AE_Regulieres (tableau 14-10-0011-01)
Prestataires de l'AE, prestations régulières, données désaisonnalisées, depuis janvier 2015.

| Colonne | Type | Description |
|---|---|---|
| Date | Date | Premier jour du mois de référence |
| Géographie | Texte | Canada, provinces et territoires |
| Détail | Texte | Prestations régulières / Prestations régulières sans gains déclarés |
| Sexe | Texte | Les deux sexes / Hommes / Femmes |
| Groupe d'âge | Texte | 15 ans et plus / 15 à 24 ans / 25 à 54 ans / 55 ans et plus |
| Prestataires | Nombre entier | Nombre de prestataires |
| Qualité | Texte | Indicateur dérivé de la colonne STATUS |

### Fait_AE_Types (tableau 14-10-0009-01)
Prestataires de l'AE selon le genre de prestations, données non désaisonnalisées, de janvier 2015 à juillet 2026.

| Colonne | Type | Description |
|---|---|---|
| Date | Date | Premier jour du mois de référence |
| Géographie | Texte | 10 provinces et 3 territoires (le Canada n'est pas inclus) |
| Type de prestation | Texte | 20 types, organisés en hiérarchie (voir section 3) |
| Prestataires | Nombre entier | Nombre de prestataires |
| Qualité | Texte | Indicateur dérivé de la colonne STATUS |

### Fait_Chomage (tableau 14-10-0287-01)
Taux de chômage, Enquête sur la population active, 15 ans et plus, données désaisonnalisées, de janvier 2015 à septembre 2026.

| Colonne | Type | Description |
|---|---|---|
| Date | Date | Premier jour du mois de référence |
| Géographie | Texte | Canada et 10 provinces (l'EPA ne couvre pas les territoires) |
| Sexe | Texte | Harmonisé : Les deux sexes / Hommes / Femmes |
| Taux de chômage | Nombre décimal | Stocké en fraction (0,068 = 6,8 %) |
| Qualité | Texte | Indicateur dérivé de la colonne STATUS |

## 3. Hiérarchies et risque de double comptage

Les fichiers mélangent des totaux et des détails dans une même colonne :
- Sexe : « Les deux sexes » = Hommes + Femmes
- Groupe d'âge : « 15 ans et plus » = somme des trois groupes
- Détail : « sans gains déclarés » est un sous-ensemble des prestations régulières
- Type de prestation : « Tous les genres » (total), 8 types principaux sans chevauchement, « Prestations spéciales » (sous-total) et des sous-types détaillés

Les mesures DAX filtrent explicitement le niveau voulu afin d'éviter tout double comptage.

## 4. Qualité des données (colonne STATUS)

| Code | Signification | Traitement |
|---|---|---|
| (vide) | Donnée fiable | Qualité = « Fiable » |
| E | À utiliser avec prudence | Conservée, signalée dans les info-bulles |
| .. | Non disponible | Valeur laissée vide (null), jamais remplacée par 0 |

## 5. Validation du chargement

Le nombre de lignes chargées a été comparé au nombre attendu, calculé à partir des dimensions de chaque fichier.

| Table | Calcul attendu | Attendu | Chargé |
|---|---|---|---|
| Fait_AE_Types | 13 géographies × 20 types × 139 mois | 36 140 | 36 140 |
| Fait_Chomage | 11 géographies × 3 genres × 139 mois | 4 653 | 4 653 |
| Fait_AE_Regulieres | Tableau complet filtré à partir de 2015 | – | 28 912 |

Constat : au premier chargement, Fait_AE_Types et Fait_Chomage comptaient chacune 2 lignes de trop, qui provoquaient 2 erreurs. Il s'agissait des lignes de pied de page ajoutées par Statistique Canada aux fichiers de « données sélectionnées ». Elles ont été éliminées par un filtre qui ne conserve que les lignes dont la période a le format « AAAA-MM ».

## 6. Limites connues

- Le tableau 14-10-0009-01 n'inclut pas le Canada : le total national est calculé par addition des provinces et territoires.
- L'EPA ne couvre pas les territoires : les comparaisons entre AE et chômage portent uniquement sur les provinces.
- Les données désaisonnalisées (tableaux A et C) et non désaisonnalisées (tableau B) ne sont jamais combinées dans un même visuel.
- Selon Statistique Canada, les catégories « Hommes+ » et « Femmes+ » de l'EPA incluent certaines personnes non binaires ; elles ont été harmonisées avec « Hommes » et « Femmes ».
- Certains types de prestations (proches aidants, parentales prolongées) n'existent que depuis la fin de 2017 : les valeurs antérieures sont vides.