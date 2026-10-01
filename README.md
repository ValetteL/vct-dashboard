# VCT Dashboard — base commune

Projet Power BI (format `.pbip`, versionnable avec Git) sur les agents et les maps du Valorant Champions Tour 2021-2026.

**Problématique** : comment la méta des agents a-t-elle évolué en compétition depuis 2021, et dans quelle mesure dépend-elle de la map jouée ?

Source des données : [Kaggle, Valorant Champion Tour 2021-2026 Data](https://www.kaggle.com/datasets/ryanluong1/valorant-champion-tour-2021-2023-data).

## Ouvrir le projet

1. Power BI Desktop : **Fichier > Options > Fonctionnalités en préversion**, vérifier que le format projet (.pbip) est activé.
2. Ouvrir `VCT.pbip`.
3. **Accueil > Transformer les données > Gérer les paramètres** : mettre `Dossier_Data` sur le dossier `data\` de votre copie du dépôt (garder le `\` final).
4. **Actualiser** : le chargement lit les CSV de `data\`, nettoie et construit le modèle.

> Pour éviter les conflits Git sur le paramètre, convenez d'un chemin commun (ex. `C:\vct-dashboard\`) ou ne commitez jamais `expressions.tmdl` après avoir changé le chemin.

## Contenu

| Dossier / fichier | Rôle |
|---|---|
| `data/` | CSV bruts utiles (3 fichiers x 6 saisons, ~44 Mo) : `maps_stats`, `teams_picked_agents`, `draft_phase` |
| `VCT.SemanticModel/` | Modèle : requêtes Power Query, relations, mesures DAX (lire les fichiers `.tmdl`) |
| `VCT.Report/` | Rapport : 4 pages vides (Overview, Évolution, Maps, Performance) |

## Modèle de données

- `Contextes` : une ligne par année, tournoi, stage, match type et map (nombre de maps jouées, win rate attaque/défense).
- `Picks` : équipe x agent x contexte (apparitions, victoires, défaites). Liée à `Contextes` par la clé `Cle`.
- `Agents` : agent, rôle (Duelliste, Initiateur, Contrôleur, Sentinelle), première année jouée.
- `Cartes`, `Annees` : dimensions pour les segments (filtrent `Contextes` et `Draft`).
- `Draft` : bans et picks de maps par match.
- `_Mesures` : toutes les mesures DAX.

**À utiliser comme segments** : `Annees[Annee]`, `Cartes[Map]`, `Agents[Role]`, `Agents[Agent]`, `Contextes[Tournoi]`.
**Ne pas filtrer sur `Picks[Equipe]`** avec `[Pick Rate]` : le dénominateur (maps jouées) ne suit pas ce filtre.

## Pièges traités dans Power Query

- Lignes agrégées `All Maps` / `All Stages` / `All Match Types` retirées (sinon tout est compté en double).
- Pourcentages en texte (`"46%"`) convertis en nombres.
- Année ajoutée depuis le nom du fichier (un nom de tournoi seul n'est pas unique entre 2021 et 2022).
- Noms d'agents mis en forme (`kayo` devient `KAY/O`).
- Aucune ligne de remplissage à 0 % (on repart de `teams_picked_agents`, pas de `agents_pick_rates`).

## Règles d'équipe

1. Une personne seulement touche à `VCT.SemanticModel/` (Power Query, relations, mesures).
2. Chacun travaille uniquement dans le dossier de **sa page** : `VCT.Report/definition/pages/<Page>/`.
3. Une branche Git par personne, fusion dans `main` après relecture.
4. Une mesure utile à une seule page est demandée à la personne responsable du modèle, ou préfixée (`B_...`) et listée dans le message de commit.
5. Pool de maps et agents changent selon les saisons : filtrer par `Cartes[PremiereAnnee]` ou `Agents[PremiereAnnee]` avant de comparer entre années.
