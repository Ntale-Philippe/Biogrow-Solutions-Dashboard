# BioGrow Solutions — Pourquoi certains lots de production sont-ils invendables ?

Tableau de bord **Power BI** sur la production de souches bactériennes de BioGrow Solutions.

## 1. Le problème

Sur la période étudiée, **16 % des lots produits sont invendables** : leur rendement est inférieur au seuil de qualité de leur souche. Cela représente **317 k€ de coûts de production perdus**, sur 1,99 M€ au total.

Questions métier :
- Quels lots deviennent invendables, et pourquoi ?
- Le problème vient-il des souches, des sites, des techniciens ou des conditions de culture ?
- Où agir en priorité pour réduire les pertes ?

## 2. Les données

**Source** : 5 fichiers CSV.

| Fichier | Contenu | Taille |
|---|---|---|
| `BioGrow_Fait_Production` | un lot par ligne : date, souche, site, technicien, température, pH, coût, rendement (UFC/ml) | 2 000 lots, janv. 2025 → mars 2026 |
| `BioGrow_Dim_Souches` | température optimale, seuil de qualité | 5 souches |
| `BioGrow_Dim_Sites` | nom, climat | 3 sites |
| `BioGrow_Dim_Techniciens` | nom, niveau d'expertise | 6 techniciens |
| `BioGrow_Dim_Calendrier` | dates, mois, trimestres | 500 jours |

**État de départ** : aucune valeur manquante ni aucun lot en double. En revanche, le format et le modèle posaient plusieurs problèmes :
- **Décimales au format anglais** (`37.5`), mal interprétées dans un Power BI configuré en français.
- **Séparateur du fichier des souches modifié** (`,` → `;`) après la création de la requête, ce qui faisait échouer l'actualisation.
- **Colonne « Mois » en texte** (`01-January`), sans l'année, alors que les données couvrent deux années.
- **Clés incohérentes** : identifiants en texte dans la table de faits, en nombres dans les dimensions.
- **Rendement arrondi à l'entier** avant la comparaison au seuil : 13 lots invendables passaient inaperçus.
- **Seuils et cibles codés en dur** dans des formules DAX, et une mesure de perte financière toujours vide à cause d'une faute de frappe dans un filtre (« Invendables » au lieu de « Invendable »).

## 3. La méthode

- **Nettoyage (Power Query)**
  - Conversion des décimales avec la bonne culture.
  - Correction du séparateur.
  - Mois recalculé à partir de la date.
  - Clés typées en nombres entiers.
  - Seuil de qualité lu dans les données plutôt que codé en dur.
- **Modélisation**
  - Schéma en étoile : une table de faits et quatre dimensions.
  - Pas d'agrégation implicite trompeuse (comme une somme de températures).
- **Règle de qualité (DAX)** : chaque lot est classé dans une des 4 catégories ci-dessous.

  | Catégorie | Condition |
  |---|---|
  | Invendable | Rendement inférieur au seuil de la souche. |
  | Premium | pH entre 6,5 et 7,5, et température à ±2 °C de l'optimum de la souche. |
  | Vendable | Rendement suffisant, mais conditions de culture hors zone idéale. |
  | À vérifier | Donnée manquante. |

- **Indicateurs (DAX)** : atteinte de la cible de rendement, taux et coût des invendables, écart à la température optimale. Des constats textuels sont recalculés à chaque actualisation.
- **Outils** : Power BI Desktop, Power Query (M), DAX, format projet PBIP/PBIR versionné avec Git. Les chiffres ont été contrôlés à part en Python, directement sur les CSV.

## 4. Le résultat

Un rapport de 5 pages : **Accueil**, **Rendement**, **Qualité**, **Efficacité commerciale** et **Recommandations**. Quatre filtres (souche, technicien, site, niveau d'expertise) sont synchronisés entre les pages.

<!-- Captures à ajouter dans docs/ :
![Page Rendement](docs/rendement.png)
![Page Qualité](docs/qualite.png)
![Page Efficacité commerciale](docs/commercial.png)
![Page Recommandations](docs/recommandations.png)
-->

Ce que montrent les données :

| Constat | Chiffres |
|---|---|
| **Une souche concentre l'essentiel des pertes** | Bacillus S : 59 % de lots invendables, soit **72 % de la perte** (230 k€). Toutes les autres souches réunies : 5 %. Son rendement moyen (98 UFC/ml) est **sous son propre seuil** (100). |
| **Un site décroche** | Station Nairobi : 21,6 % d'invendables et un rendement moyen de 90 UFC/ml, contre 13 à 15 % et 99 UFC/ml ailleurs. Le coût par lot est pourtant identique (≈ 1 000 €). |
| **Une souche excelle** | Pediococcus A : 104 % de la cible de rendement, aucun lot invendable. |
| **L'ancienneté n'explique pas la qualité** | Les deux techniciens juniors ont les taux d'invendables les plus bas (11 et 14 %). Le coût par lot est quasi identique d'un niveau d'expertise à l'autre (985 à 1 010 €). |
| **Le problème est permanent** | Entre 13 et 28 lots invendables chaque mois, sans aucun mois épargné. |

## 5. Recommandations et impact

1. **Traiter Bacillus S en priorité.** C'est le levier le plus rentable : si son taux d'invendables rejoignait celui des autres souches (5 %), on éviterait de l'ordre de **200 k€ de pertes** sur la période.
   - Revoir le procédé de production.
   - Vérifier que le seuil (100 UFC/ml) et la cible (140 UFC/ml) sont réalistes.
   - Éviter les cultures sous 36 °C (71 % d'invendables, contre 55 à 57 % entre 36 et 40 °C).
2. **Auditer les pratiques de la Station Nairobi.** Le coût par lot y est le même qu'ailleurs, donc le problème n'est pas budgétaire : il vient du procédé ou de l'équipement. 84 % de ses lots de Bacillus S sont invendables.
3. **Développer Pediococcus A** dans le catalogue, la seule souche sans aucune perte.
4. **Suivre la qualité en continu.** Des invendables apparaissent chaque mois : un suivi en temps réel au laboratoire permettrait d'écarter plus tôt les lots non conformes.
5. **Cibler l'accompagnement sur les résultats individuels plutôt que sur le niveau d'expertise.** Les écarts entre techniciens de même niveau (14 à 18 % chez les seniors) sont plus grands que les écarts entre niveaux.

## Ouvrir le projet

1. Ouvrez `Biogrow dashboard.pbip` dans une version récente de Power BI Desktop.
2. Les CSV ne sont pas inclus dans le dépôt. Indiquez leur emplacement via **Transformer les données → Paramètres de la source de données**, puis cliquez sur **Actualiser**.

Les icônes de navigation sont dérivées de [Lucide](https://lucide.dev) (licence ISC, voir [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)).

*Auteur : Philippe Ntale*
