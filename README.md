# VCT Dashboard — base commune

Projet Power BI (format `.pbip`, versionnable avec Git) sur les agents et les maps du Valorant Champions Tour 2021-2026.

**Problématique** : comment la méta des agents a-t-elle évolué en compétition depuis 2021, et dans quelle mesure dépend-elle de la map jouée ?

Source des données : [Kaggle, Valorant Champion Tour 2021-2026 Data](https://www.kaggle.com/datasets/ryanluong1/valorant-champion-tour-2021-2023-data).

## Ouvrir le projet

1. Power BI Desktop : **Fichier > Options > Fonctionnalités en préversion**, vérifier que le format projet (.pbip) est activé.
2. Ouvrir `VCT.pbip`.
3. **Accueil > Transformer les données > Gérer les paramètres** : mettre `Dossier_Data` sur le dossier `data\` de votre copie du dépôt (garder le `\` final).
4. **Actualiser** : le chargement lit les CSV de `data\`, nettoie et construit le modèle.

> Le chemin `Dossier_Data` est propre à chaque PC : on ne renvoie jamais ce fichier (voir « Comment travailler à quatre »).

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

## Thème et couleurs

Le thème commun est dans `theme/VCT_Theme.json` (importé dans le rapport : **Affichage > Thèmes**).

| Usage | Couleur |
|---|---|
| Contrôleur | `#7B61FF` (violet) |
| Duelliste | `#FF6B35` (orange) |
| Initiateur | `#F4B400` (ambre) |
| Sentinelle | `#14A3B8` (cyan) |
| Données « normales » | `#2D3A4B` (ardoise) |
| Données secondaires | `#9AA5B1` (gris) |
| Positif | `#2E9E5B` (vert) |
| Alerte | `#E63946` (rouge) |

**Colorer par rôle** : sur un graphique, **Format > Couleurs > fx > Format selon : Valeur du champ**, puis choisir la mesure `[Couleur Rôle]`. Ne pas se fier à l'ordre automatique des couleurs : il change si un rôle est filtré.
Pour une carte de chaleur (matrice agent x map), utiliser l'échelle claire vers foncée du thème (minimum / centre / maximum).

## Comment travailler à quatre

Chacun travaille **dans sa propre copie** du projet, sans Git obligatoire : la copie sert de bac à sable. À la fin, on ne renvoie que **sa page** et **sa table de mesures**.

1. **Récupérer le projet** : `git clone` ou *Code > Download ZIP* sur GitHub, puis suivre « Ouvrir le projet » ci-dessus.
2. **Construire sa page** (Overview, Évolution, Maps ou Performance). Les mesures communes (`_Mesures`) se réutilisent telles quelles.
3. **Ses mesures en plus** vont dans **sa table** : `_Mesures_Overview`, `_Mesures_Evolution`, `_Mesures_Maps` ou `_Mesures_Performance` (clic droit sur la table > Nouvelle mesure).
4. **Enregistrer**, vérifier que le projet s'ouvre et s'actualise sans erreur.
5. **Renvoyer uniquement ces deux éléments** (en ZIP, par Teams / Discord / mail) :
   - le dossier de sa page : `VCT.Report/definition/pages/<Page>/`
   - le fichier de ses mesures : `VCT.SemanticModel/definition/tables/_Mesures_<Page>.tmdl`
6. La personne qui gère le dépôt les copie dans le projet principal, ouvre, actualise, et publie dans `main`.

**Ne pas renvoyer** : `expressions.tmdl` (chemin de votre PC), `model.tmdl`, les autres tables, `report.json`.
Il manque une colonne, une table ou une relation ? Le dire à celui qui gère le modèle plutôt que de la créer dans sa copie.

Pool de maps et agents changent selon les saisons : filtrer par `Cartes[PremiereAnnee]` ou `Agents[PremiereAnnee]` avant de comparer entre années.
