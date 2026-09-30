# Réussite scolaire au Portugal

Analyse exploratoire et tableau de bord consacrés à la réussite scolaire dans deux lycées portugais, à partir des résultats en mathématiques et en portugais. Le projet combine un notebook Python, deux fichiers CSV d’origine et un rapport Power BI.

> **Statut :** analyse exploratoire/documentation — aucune modélisation prédictive n’est implémentée dans le notebook fourni.

## Sommaire

- [Objectif](#objectif)
- [Contexte et sources](#contexte-et-sources)
- [Fichiers](#fichiers)
- [Données et schéma](#données-et-schéma)
- [Méthodologie](#méthodologie)
- [Résultats et insights](#résultats-et-insights)
- [Technologies](#technologies)
- [Reproduire l’analyse](#reproduire-lanalyse)
- [Limites et points de vigilance](#limites-et-points-de-vigilance)
- [Pistes d’approfondissement](#pistes-dapprofondissement)

## Objectif

Le projet cherche à décrire les facteurs associés aux notes finales et à comparer les parcours en mathématiques et en portugais. Il vise notamment à :

- explorer le profil scolaire, familial et social des élèves ;
- comparer les résultats selon la matière, l’établissement, le lieu de résidence et le sexe ;
- rapprocher les deux fichiers par élève afin de construire une table d’analyse commune ;
- qualifier la situation finale (`Réussite`, `Échec` ou `Abandon`) à partir de la note du troisième trimestre `G3` ;
- produire des indicateurs et des visualisations utilisables dans Power BI.

La « réussite » est définie dans le notebook par `G3 >= 10`. La valeur `G3 == 0` est traitée séparément comme un **abandon** ; les notes strictement inférieures à 10 et différentes de 0 sont classées en **échec**.

## Contexte et sources

Les supports fournis présentent les données comme issues de l’étude de **P. Cortez et A. Silva (2008)**, Université du Minho, sur deux lycées publics de la région de l’Alentejo, pour l’année scolaire **2005–2006**. Les données associent des réponses à un questionnaire et des résultats issus des bulletins scolaires.

Les fichiers locaux contiennent :

- `student-mat.csv` : 395 lignes et 33 colonnes pour la matière mathématiques ;
- `student-por.csv` : 649 lignes et 33 colonnes pour la matière portugais.

Les deux fichiers ne représentent pas nécessairement exactement les mêmes élèves : le notebook effectue donc une fusion externe (`outer`) sur un ensemble de caractéristiques communes, et non sur un identifiant source explicitement fourni.

## Fichiers

| Fichier | Rôle |
|---|---|
| `Notebook_final.ipynb` | Chargement, exploration, préparation, tests statistiques et création des catégories de motivation. |
| `student-mat.csv` | Données de résultats et de contexte en mathématiques ; séparateur `;`. |
| `student-por.csv` | Données de résultats et de contexte en portugais ; séparateur `;`. |
| `Reussite_scolaire.pptx` | Présentation du contexte, de la démarche et des limites/pistes futures. |
| `Reussite scolaire au Portugal.pbix` | Rapport Power BI fourni. Son modèle binaire n’a pas été décompressé ni interprété au-delà des métadonnées lisibles. |

Le notebook écrit également un fichier intermédiaire `data_final.csv` lors de son exécution, mais ce fichier n’est pas présent parmi les fichiers fournis.

## Données et schéma

### Schéma des CSV bruts

Chaque ligne correspond à un enregistrement élève–matière. Les deux CSV partagent les 33 colonnes suivantes :

- **Identification et profil :** `school`, `sex`, `age`, `address`, `famsize`, `Pstatus` ;
- **Contexte familial et orientation :** `Medu`, `Fedu`, `Mjob`, `Fjob`, `reason`, `guardian` ;
- **Scolarité et accompagnement :** `traveltime`, `studytime`, `failures`, `schoolsup`, `famsup`, `paid`, `activities`, `nursery`, `higher`, `internet` ;
- **Vie sociale et santé :** `romantic`, `famrel`, `freetime`, `goout`, `Dalc`, `Walc`, `health`, `absences` ;
- **Évaluations :** `G1`, `G2`, `G3`.

Les variables catégorielles utilisent notamment des codes courts (`F/M`, `U/R`, `yes/no`) et les variables ordinales sont codées numériquement. Les notes `G1`, `G2` et `G3` sont sur une échelle observée allant de 0 à 20 ; `G3` est la note finale utilisée pour la qualification de réussite.

### Table préparée par le notebook

Le notebook :

1. ajoute une colonne de matière (`cours = maths` ou `portugais`) ;
2. renomme les colonnes en français ;
3. fusionne les fichiers sur 23 attributs communs, avec suffixes `_math` et `_por` pour les variables propres à chaque matière ;
4. obtient 674 lignes et 45 colonnes après la fusion ;
5. ajoute `cle_primaire`, une clé construite par concaténation de 15 attributs ;
6. transforme ensuite la table en 674 lignes et 46 colonnes ;
7. ajoute notamment `reussite_echec_math` et `reussite_echec_por`.

La table préparée contient notamment :

- les attributs de profil renommés (`etablissement`, `sexe`, `age`, `type_domicile`, `taille_famille`, etc.) ;
- les variables de contexte par matière, par exemple `temps_etude_hebdomadaire_math` et `temps_etude_hebdomadaire_por` ;
- les notes `G1_math`, `G2_math`, `G3_math`, `G1_por`, `G2_por`, `G3_por` ;
- les absences par matière (`nombre_absences_math`, `nombre_absences_por`) ;
- les statuts de réussite par matière.

Les libellés de plusieurs modalités sont francisés : sexe, zone de résidence, taille de la famille, statut des parents, niveaux d’éducation, professions, temps de trajet et temps d’étude. Le notebook conserve toutefois certaines variables numériques pour faciliter leur usage dans Power BI.

## Méthodologie

### Préparation et contrôle

- Import avec `pandas.read_csv(..., sep=';')`.
- Inspection des premières/dernières lignes, des types, statistiques descriptives et doublons.
- Les CSV fournis ne contiennent aucune valeur manquante ni doublon exact selon le contrôle du notebook.
- Fusion externe sur des caractéristiques communes, puis génération d’une clé composite.
- Transformation des codes en libellés lisibles.
- Création des statuts de réussite à partir de `G3`.

### Analyse

- Matrice de corrélation des variables numériques, visualisée sous forme de heatmap avec `seaborn`.
- Tests t de Welch (`scipy.stats.ttest_ind(..., equal_var=False)`) pour comparer des moyennes entre groupes.
- Création d’une catégorisation de motivation à partir de règles combinant statut de réussite, temps d’étude et évolution entre `G1` et `G3`. Les catégories observées dans la sortie du notebook sont : `Standard` (483), `Pas motivé en réussite` (63), `Démotivé` (63), `Motivé` (41) et `Motivé en difficulté` (24).
- Aucun modèle de machine learning, entraînement/test, validation croisée ou métrique de prédiction n’est présent dans le notebook inspecté.

## Résultats et insights

Les chiffres ci-dessous reprennent les sorties enregistrées dans le notebook. Ils décrivent des différences observées ; ils ne démontrent pas un lien causal.

### Statistiques descriptives

| Matière | N | Moyenne `G3` | Médiane | Minimum–maximum |
|---|---:|---:|---:|---:|
| Mathématiques | 395 | 10,42 | 11 | 0–20 |
| Portugais | 649 | 11,91 | 12 | 0–19 |

Le test t comparant les notes finales des deux matières donne `p = 2,215 × 10⁻⁸` dans le notebook : la différence de moyenne est statistiquement significative dans cet échantillon.

### Comparaisons de groupes

- **Établissement :** en mathématiques, la moyenne affichée est de 10,49 pour GP contre 9,85 pour MS (`p = 0,3431`) ; en portugais, 12,58 contre 10,65 (`p = 6,212 × 10⁻¹¹`).
- **Lieu de résidence :** en mathématiques, 9,51 en zone rurale contre 10,67 en zone urbaine (`p = 0,0366`) ; en portugais, 11,09 contre 12,26 (`p = 7,274 × 10⁻⁵`).
- **Sexe :** en mathématiques, 9,97 pour les filles contre 10,91 pour les garçons (`p = 0,0396`) ; en portugais, 12,25 contre 11,41 (`p = 0,0011`).
- **Personas de motivation :** la sortie indique notamment 9,31 pour les élèves « motivés en difficulté » contre 2,37 pour les « démotivés » en mathématiques (`p` affichée à 0), tandis que la comparaison entre « pas motivé en réussite » et « motivé » en mathématiques affiche `p = 0,1483`.

Ces comparaisons doivent être lues avec prudence : les tailles de groupes, la construction de la fusion et la non-indépendance possible des observations peuvent influencer les résultats.

## Technologies

- **Python** : langage d’analyse ;
- **Jupyter Notebook** : exécution et traçabilité de la démarche ;
- **pandas** et **NumPy** : manipulation, préparation et calculs ;
- **Matplotlib** et **Seaborn** : visualisations, notamment la heatmap de corrélation ;
- **SciPy** : tests t de Welch ;
- **Power BI** : rapport fourni (`.pbix`) et exploration visuelle.

L’inspection ZIP/XML du PBIX confirme un rapport contenant **7 pages**, un thème personnalisé et des visuels personnalisés de type box-and-whisker. Les métadonnées seules ne permettent pas de documenter de manière fiable le détail de chaque mesure, relation, filtre ou visuel ; ces éléments ne sont donc pas décrits ici comme des faits établis.

## Reproduire l’analyse

1. Placer `student-mat.csv` et `student-por.csv` dans le même répertoire que le notebook, ou adapter les chemins `/content/...` présents dans les cellules.
2. Installer les dépendances :

   ```bash
   pip install pandas numpy matplotlib seaborn scipy jupyter
   ```

3. Ouvrir puis exécuter `Notebook_final.ipynb` de haut en bas :

   ```bash
   jupyter notebook Notebook_final.ipynb
   ```

4. Vérifier les sorties statistiques et la génération de `data_final.csv`.
5. Ouvrir le fichier PBIX avec une version compatible de Power BI Desktop si le tableau de bord doit être consulté.

Pour une reproductibilité robuste, il est recommandé de fixer les versions Python/packages et de remplacer les chemins absolus par des chemins relatifs au projet.

## Limites et points de vigilance

- **Représentativité :** l’étude porte sur deux établissements d’une région et sur des données anciennes (2005–2006). Les résultats ne sont pas généralisables à l’ensemble du Portugal ni aux élèves actuels.
- **Fusion sans identifiant source :** la clé composite est construite après une fusion sur des attributs communs. Le notebook signale trois valeurs dupliquées pour cette clé ; l’unicité d’un élève ne peut donc pas être garantie.
- **Échantillons différents :** les effectifs mathématiques et portugais diffèrent (395 contre 649). La comparaison des matières n’est pas nécessairement appariée élève par élève.
- **Variables auto-déclarées et notes scolaires :** plusieurs facteurs viennent d’un questionnaire, tandis que les notes sont des évaluations d’enseignants ; les biais de mesure et de déclaration sont possibles.
- **Abandons :** `G3 == 0` est interprété comme « Abandon » selon le notebook. Cette convention doit être confirmée par la documentation originale avant toute décision métier.
- **Tests multiples :** de nombreux tests t sont exécutés sans correction explicitement appliquée pour comparaisons multiples et sans intervalle de confiance ni taille d’effet.
- **Anomalies de code :** les cellules 60 et 61 affichent des tests sur les absences mais réutilisent les variables `motive` et `demotive` au lieu de `abs_motiv`, `abs_demo`, `abs_MF` et `abs_MR` dans l’appel au test. Les p-values et conclusions correspondantes ne doivent pas être utilisées sans correction et réexécution. Les cellules 44 et 45 contiennent aussi une expression d’affichage mal formée autour de l’arrondi de la p-value.
- **Personas heuristiques :** les catégories de motivation sont des règles d’interprétation définies dans le notebook, pas une mesure validée psychométriquement.
- **PBIX :** le fichier est binaire et son modèle interne n’a pas été interprété exhaustivement ; les détails non vérifiables ne sont pas affirmés dans ce README.
- **Données sensibles :** les variables décrivent des caractéristiques scolaires, familiales, sociales et de santé. Éviter toute ré-identification, diffusion inutile ou décision individuelle automatisée.

## Pistes d’approfondissement

Les supports proposent notamment de :

1. élargir l’échantillon à plusieurs régions et établissements ;
2. répéter l’étude pour mesurer l’évolution sur environ 20 ans ;
3. actualiser les usages numériques (temps d’écran, smartphone, réseaux sociaux, IA, apprentissage en ligne) ;
4. ajouter le sommeil, la santé mentale, le stress et le projet professionnel.

Avant toute modélisation, il serait également pertinent de définir un identifiant ou une règle d’appariement documentée, corriger les cellules de tests statistiques, distinguer clairement les observations appariées des observations indépendantes et ajouter des contrôles de qualité automatisés.

## Références

- P. Cortez et A. Silva (2008), étude sur la performance scolaire au Portugal, citée dans les supports fournis.
- Fichiers et résultats inspectés dans ce dépôt : `student-mat.csv`, `student-por.csv`, `Notebook_final.ipynb`, `Reussite_scolaire.pptx` et `Reussite scolaire au Portugal.pbix`.
