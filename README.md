# BioGrow Solutions — Rapport du secteur de production

Tableau de bord **Power BI** qui analyse la production de souches bactériennes de BioGrow Solutions : rendement des lots, qualité, coûts de production, et recommandations.

Le rapport est enregistré au **format projet Power BI (.pbip / PBIR)**. Les pages, les visuels, le thème et le modèle sémantique sont donc des fichiers texte (JSON et TMDL) lisibles et versionnables dans git.

---

## Pages du rapport

| Page | Contenu |
|---|---|
| **Accueil** | Titre, logo et accès aux analyses. |
| **Rendement** | Atteinte de la cible, rendement moyen et écart-type ; évolution mensuelle du rendement ; atteinte de la cible par souche (vert si la cible est atteinte, corail sinon). |
| **Qualité** | pH moyen, température moyenne et écart à la température optimale des souches ; évolution mensuelle du rendement et de la température comparée à l'optimum. |
| **Efficacité commerciale** | Coût de production, perte financière due aux lots invendables, nombre de lots invendables ; coûts par niveau d'expertise et par site. |
| **Recommandations** | Six recommandations, chacune accompagnée d'un « Constat actuel » recalculé à chaque actualisation des données. |

Sur les trois pages d'analyse, quatre segments déroulants sont **synchronisés entre les pages** : souche, technicien, site et niveau d'expertise. Un bouton **« Réinitialiser les filtres »** efface toutes les sélections.

## Modèle de données

Schéma en étoile, importé depuis cinq fichiers CSV :

```
                 BioGrow_Dim_Calendrier
                          │
BioGrow_Dim_Souches ── BioGrow_Fait_Production ── BioGrow_Dim_Sites
                          │
                 BioGrow_Dim_Techniciens
```

- **BioGrow_Fait_Production** : un lot par ligne (date, souche, site, technicien, température, pH, coût, rendement en UFC/ml).
- **BioGrow_Dim_Souches** : nom, température optimale, seuil de qualité, rendement cible.
- **BioGrow_Dim_Sites**, **BioGrow_Dim_Techniciens** (avec le niveau d'expertise), **BioGrow_Dim_Calendrier**.
- **Mesures** : table dédiée aux mesures DAX.

### Règle de qualité d'un lot (`Categorie_Qualite`)

| Catégorie | Condition |
|---|---|
| **À vérifier** | La souche n'a pas de seuil ou de température optimale, ou le lot n'a pas de rendement. |
| **Invendable** | Le rendement est inférieur au seuil de qualité de la souche. |
| **Premium** | Le rendement atteint le seuil, le pH est entre 6,5 et 7,5, et la température est à ±2 °C de l'optimum de la souche. |
| **Vendable** | Le rendement atteint le seuil, mais les conditions de culture sont hors zone idéale. |

### Principales mesures

| Mesure | Rôle |
|---|---|
| `Performance par souche en %` | Atteinte de la cible : rendement du lot divisé par le rendement cible de la souche, en moyenne (100 % = cible atteinte). |
| `Nombre de Lots invendables` / `Taux de lots invendables` | Lots classés « Invendable », en nombre et en part de la production. |
| `Perte Financière (Invendables)` | Coût de production des lots invendables. |
| `Cout de production total` / `Coût moyen par lot` | Coûts de production (affichés en M€ et en €). |
| `Température optimale` / `Écart à la température optimale` | Optimum moyen des souches produites et écart à la température réelle. |
| Dossier **Constats** | Mesures texte (souche la plus et la moins performante, site le plus coûteux, etc.) affichées sur la page Recommandations. |

> Certaines mesures gardent une orthographe d'origine (`Rendemant moyen par lot`, `Cout de production total`, `temperature moyenne`, `ph moyen`). Elles ne sont pas renommées, pour ne casser aucune référence ; les visuels affichent des libellés corrigés.

## Charte graphique

| Rôle | Couleur |
|---|---|
| Principale (sarcelle du logo) | `#1F8A84` |
| Accent (orange du logo, élément actif) | `#F2A65A` |
| Positif / alerte | `#3E9B6E` / `#E76F51` |
| Texte principal / secondaire | `#2F3E46` / `#6B7A80` |
| Fond de page / cartes / bordures | `#F4F2ED` / `#FFFFFF` / `#E6E2DA` |

- Police : Segoe UI (titres en Segoe UI Semibold).
- Les cartes blanches ont des coins arrondis de 12 px et une ombre légère.
- Canevas 16:9 (1280 × 720) sur une grille régulière, avec des marges de 16 px.
- Le thème est défini dans `StaticResources/RegisteredResources/BioGrowTheme.json`.
- La barre de navigation utilise des icônes SVG (dérivées de [Lucide](https://lucide.dev)), avec un état au survol et la page active surlignée en orange.

## Structure du dépôt

```
Biogrow dashboard.pbip                  ← fichier à ouvrir dans Power BI Desktop
Biogrow dashboard.Report/
  definition/
    report.json                         ← configuration du rapport et du thème
    pages/<page>/page.json              ← une page
    pages/<page>/visuals/<id>/visual.json  ← un visuel
  StaticResources/RegisteredResources/  ← thème, icônes SVG, logos
Biogrow dashboard.SemanticModel/
  definition/
    tables/*.tmdl                       ← tables, colonnes, mesures DAX, requêtes Power Query
    relationships.tmdl                  ← relations du modèle
```

## Ouvrir le projet

1. Installez une version récente de **Power BI Desktop** (format projet PBIR pris en charge).
2. Clonez le dépôt, puis ouvrez `Biogrow dashboard.pbip`.
3. **Les données ne sont pas incluses dans ce dépôt.** Les requêtes lisent cinq fichiers CSV par chemin absolu (`C:\Users\Admin\Downloads\BioGrow_*.csv`) :
   - `BioGrow_Fait_Production.csv`
   - `BioGrow_Dim_Souches.csv`
   - `BioGrow_Dim_Sites.csv`
   - `BioGrow_Dim_Techniciens.csv`
   - `BioGrow_Dim_Calendrier.csv`

   Pour utiliser vos propres fichiers, ouvrez **Transformer les données → Paramètres de la source de données** et indiquez leur emplacement.
4. Cliquez sur **Actualiser**. Le cache de données (`.pbi/cache.abf`) n'est pas versionné, donc le rapport s'ouvre vide jusqu'à la première actualisation.

## Auteur

Philippe Ntale

## Crédits

Les icônes de navigation sont dérivées de [Lucide](https://lucide.dev), sous licence ISC. Voir [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
