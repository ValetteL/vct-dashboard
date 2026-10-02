# Page Maps : passation pour la mise en commun

Tout ce qu'il faut pour reproduire la page **Maps** dans un autre rapport. Source de vérité : ce dépôt (le `.pbix` envoyé est une copie).

## Le plus simple : partir de ce dépôt comme rapport principal

Le dossier contient déjà la page Maps, le modèle v2, le thème et le Chiclet. Pour fusionner :

1. Prendre ce dépôt comme base (`git pull` ou ZIP).
2. Y copier les pages des autres (`VCT.Report/definition/pages/<Page>/`) et leurs fichiers de mesures (`VCT.SemanticModel/definition/tables/_Mesures_<Page>.tmdl`).
3. Ouvrir `VCT.pbip`, Actualiser, vérifier (voir la fin).

Les tables `_Mesures_Overview`, `_Mesures_Evolution`, `_Mesures_Performance` existent déjà dans ce modèle (vides) : il suffit de remplacer leur fichier par celui du collègue.

## Si on reproduit plutôt dans un autre rapport (base v1)

La page Maps n'a besoin, côté modèle, que d'**une seule colonne en plus de la v1 : `Cartes[Banniere]`** (image de chaque map).

1. **Données** : copier `data/cartes_visuels.csv` dans le dossier `data\` du rapport.
2. **Modèle** (dans `VCT.SemanticModel/definition/`) :
   - copier `tables/CartesVisuels.tmdl` (nouveau) ;
   - remplacer `tables/Cartes.tmdl` par celui-ci (il fusionne `Splash`, `Minimap`, `Banniere`) ;
   - copier `tables/_Mesures_Maps.tmdl` (les mesures de la page) ;
   - dans `model.tmdl`, ajouter la ligne `ref table CartesVisuels` avec les autres `ref table`.
3. **Visuel personnalisé Chiclet Slicer** : l'installer depuis AppSource (Accueil > Plus de visuels > Depuis AppSource) **avant** d'ouvrir la page, ou ajouter dans `VCT.Report/definition/report.json` la clé `"publicCustomVisuals": ["ChicletSlicer1448559807354"]`.
4. **Page** : copier tout le dossier `VCT.Report/definition/pages/Maps/` (la mise en forme, les interactions entre visuels, les filtres Top 10 et les règles de couleur y sont enregistrés).
5. **Thème** : Affichage > Thèmes > Parcourir > `theme/VCT_Dark.json` (le thème s'applique à tout le rapport : à décider en groupe).
6. Ouvrir, **Actualiser**, vérifier.

## Mesures

### Déjà dans la base v1 (table `_Mesures`) : la page en dépend

| Mesure | DAX |
|---|---|
| `Maps jouées` | `SUM(Contextes[MapsJouees])` |
| `Nb apparitions` | `SUM(Picks[Apparitions])` |
| `Pick Rate` | `DIVIDE([Nb apparitions], 2 * [Maps jouées])` |
| `Agents par équipe` | `[Pick Rate]` |
| `Win Rate Attaque` | `DIVIDE(SUMX(Contextes, Contextes[WinAttaque] * Contextes[MapsJouees]), [Maps jouées])` |

### Créées pour la page Maps (table `_Mesures_Maps`)

| Mesure | Utilisée par | DAX |
|---|---|---|
| `Avantage attaque (pts)` | graphique d'avantage attaque / défense | `([Win Rate Attaque] - 0.5) * 100` |
| `Écart de pick vs toutes maps (pts)` | « Ce qui distingue la map » | `([Pick Rate] - CALCULATE([Pick Rate], REMOVEFILTERS(Cartes))) * 100` |
| `Écart absolu (pts)` | filtre Top 10 du même visuel | `ABS([Écart de pick vs toutes maps (pts)])` |
| `Part de compo qui change selon la map` | KPI | voir ci-dessous |
| `Statut de la map` | KPI | voir ci-dessous |
| `Agent n°1` | KPI | `MAXX(TOPN(1, VALUES(Agents[Agent]), [Pick Rate], DESC), Agents[Agent])` |
| `Titre agents distinctifs` | titre dynamique | `"Ce qui distingue " & SELECTEDVALUE(Cartes[Map], "les maps") & " (écart de pick, pts)"` |

```dax
Part de compo qui change selon la map =
VAR total = CALCULATE([Agents par équipe], REMOVEFILTERS(Agents))
RETURN
    DIVIDE(
        SUMX(VALUES(Agents[Agent]),
             ABS([Pick Rate] - CALCULATE([Pick Rate], REMOVEFILTERS(Cartes)))),
        2 * total
    )

Statut de la map =
IF(
    ISBLANK([Maps jouées]),
    "Hors pool sur cette période",
    "En jeu : " & FORMAT([Maps jouées], "0") & " maps"
)
```

### Présentes dans `_Mesures_Maps` mais pas utilisées par la page (bonus)

| Mesure | DAX |
|---|---|
| `Part des maps jouées` | `DIVIDE([Maps jouées], CALCULATE([Maps jouées], REMOVEFILTERS(Cartes)))` |
| `Écart de part de bans vs moyenne (pts)` | `([Part de bans] - CALCULATE([Part de bans], REMOVEFILTERS(Cartes))) * 100` |

(`Part de bans` est dans `_Mesures`, base v1.)

## Champs du modèle utilisés par la page

`Cartes[Map]`, **`Cartes[Banniere]`** (nouveau), `Agents[Role]`, `Agents[Agent]`, `Annees[Annee]`, et indirectement `Contextes[MapsJouees]`, `Contextes[WinAttaque]`, `Picks[Apparitions]`.

## Réglages enregistrés dans la page (rien à refaire)

- Interactions : le Chiclet (map) ne filtre ni le graphique d'avantage attaque / défense ni la composition par rôle.
- « Ce qui distingue la map » : filtre Top 10 sur `Agent` par `Écart absolu (pts)`, règles de couleur (crème au-dessus de la moyenne, gris en dessous).
- Graphique d'avantage : règles de couleur (rouge attaque, bleu défense).
- Valeurs par défaut des segments (à régler sur 2025 et Haven avant de livrer).

## Contrôles après reproduction

| Où | Attendu (Année 2025, map Haven) |
|---|---|
| Chiclet | 12 maps avec leur image |
| `Statut de la map` | « En jeu : » suivi d'un nombre de maps |
| `Agent n°1` | un agent (pas vide) |
| « Ce qui distingue Haven » | Sova, Omen, Cypher au-dessus de la moyenne |
| Composition par rôle | une colonne par map, d'environ 5 agents au total |

Une map hors pool (par exemple Sunset en 2026) donne `(Vide)` : c'est normal, pas une erreur.

## Limites à connaître

- Les parts de bans (visuel bonus) ne portent que sur les matchs où la draft est renseignée (environ la moitié des compétitions).
- Les images viennent de valorant-api.com (© Riot Games) et exigent une connexion Internet.
