# Fiche de suivi des projets agents IA

Master • Présentation de l’avancement et plan de correction

Groupe : 5IADG3   Projet n° : 25   Date : 20 Novembre 2026

Intitulé du projet : Dashboard de Pilotage des Services Publics avec Analyse Prédictive

Membres : Khadija El Hassani et Hiba Sodqi

Jalon examiné : Préparation des données et pipeline initial (semaines 4-5 du planning)

Prochain point : 

## Utilisation de la fiche

Avant chaque point, chaque groupe complète les réponses et rassemble les preuves. Présenter en 10 minutes : bilan, démonstration, résultats et blocages. Prévoir ensuite 5 minutes de questions. Chaque membre explique sa contribution et un choix technique.

Pour chaque rubrique, distinguer ce qui est prévu, commencé, démontré ou bloqué. Une fonction est démontrée lorsqu’un scénario reproductible et son résultat peuvent être vérifiés. Justifier toute question sans objet. Si un élément manque, indiquer la correction et son échéance.

## Bilan et périmètre

**1. Quel problème métier traitez-vous, pour quel utilisateur et avec quel résultat attendu ?**

Preuve à présenter : Un cas d’usage précis et un critère de réussite mesurable.

Réponse du groupe : Les services municipaux (déchets, eau, électricité) sont suivis séparément, sans vue d’ensemble ni anticipation de la demande. Notre projet est un dashboard de pilotage destiné au gestionnaire des services municipaux. Il regroupe les KPI par quartier et prévoit la demande d’un service précis afin de mieux planifier les moyens. Il ne prend aucune décision publique automatiquement : l’humain décide. Critère de réussite mesurable : sur le jeu de test, l’erreur de prévision à 7 jours (MAE) du modèle est inférieure à celle de la baseline simple. État : critère défini et baselines mesurées ; le service à prévoir reste à choisir.

**2. Quel périmètre MVP avez-vous retenu et quelles fonctions avez-vous reportées ?**

Preuve à présenter : Liste des fonctions incluses et exclues, validée par le groupe.

Réponse du groupe : MVP imposé par le sujet 25 : KPI municipaux synthétiques et prévision d’un service précis.

Inclus, déjà réalisé : découpage de la ville en 263 zones avec carte, KPI quotidiens synthétiques (2023-2025), base PostgreSQL, baselines et protocole d’évaluation.

Inclus, à réaliser : modèle de prévision (Prophet, statsmodels ou scikit-learn), API (FastAPI ou Spring Boot), dashboard Angular avec Apache ECharts, suivi Grafana.

Reporté (extension facultative) : scénarios de planification.

**3. Qu’aviez-vous prévu depuis le dernier point et qu’avez-vous réellement terminé ?**

Preuve à présenter : Fonctions terminées, en cours et bloquées, avec preuve pour chacune.

Réponse du groupe : Terminé : préparation des zones (script 01, 263 zones, GeoJSON) ; génération des KPI synthétiques (script 02, 288 248 lignes) ; base PostgreSQL chargée (comptages 288 248, 265 et 120) ; baselines et mesure de la MAE (script 03).

En cours : aucun.

Non commencé : modèle Prophet, API, dashboard, tests pytest, rapport.

Bloqué : rien. Les problèmes d’installation sous Windows (PostgreSQL absent, bash indisponible) ont été résolus. 

**4. Que peut-on exécuter aujourd’hui sans intervention manuelle cachée ?**

Preuve à présenter : Démonstration de bout en bout sur un scénario reproductible.

Réponse du groupe :La chaîne de données complète : fichiers bruts → scripts Python 01, 02, 03 → CSV et GeoJSON → chargement dans PostgreSQL → requête SQL sur la vue v\_kpi\_city\_daily. Elle est reproductible : même résultat à chaque exécution et même MAE sur deux machines (Linux et Windows). Seules étapes manuelles : placer les fichiers bruts dans data/raw/ et installer PostgreSQL. Le parcours utilisateur (API puis dashboard) n’est pas encore disponible.

État général : conforme au jalon pour la partie données.

Décision ou aide attendue de l’enseignant : valider l’usage de données synthétiques pour le MVP et le choix du service à prévoir.Données et architecture

**5. Quel dataset ou corpus utilisez-vous exactement et comment y accède-t-on ?**

Preuve à présenter : Nom, URL, version ou date, licence et fichiers retenus.

* Réponse du groupe : **Référence géographique (réelle)** : NYC Taxi \& Limousine Commission, `taxi\\\_zone\\\_lookup.csv` (265 zones) et shapefile `taxi\\\_zones` (263 polygones). Page source citée par le dictionnaire de données : http://www.nyc.gov/html/tlc/html/about/trip\_record\_data.shtml. Date des fichiers : fichiers fournis. **Licence : à vérifier.**
* **Dictionnaire de données HVFHS** (version du 18 mars 2025) : consulté mais **non exploité**, car les fichiers de courses ne sont pas téléchargés.
* **KPI (synthétiques)** : générés par `scripts/02\\\_generate\\\_synthetic\\\_kpis.py` (graine 42), du 2023-01-01 au 2025-12-31, 263 zones, 288 248 lignes.
* Accès : `data/raw/` (fichiers sources), `data/processed/` (sorties), base `pfa25`, schéma `pfa`.

**6. Les données couvrent-elles réellement le problème posé ? Que simulez-vous ?**

Preuve à présenter : Variables utiles, labels, limites et distinction entre réel et synthétique.

* Réponse du groupe : **Réel** : découpage en zones, arrondissements, géométries, jours fériés américains (calculés par règles).
* **Synthétique** : tous les KPI (déchets en tonnes, eau en m³, électricité en MWh, demandes de service, taux de collecte, délai de réponse), la population par zone (proxy surface × densité, normalisée à 8,3 M), la météo, les 39 événements et les 120 anomalies.
* **Labels** : `anomaly\\\_labels.csv` (pannes électriques, fuites, tournées manquées).
* **Limites** : aucune donnée municipale réelle ; la population par zone n’est pas réaliste zone par zone ; les effets (saisonnalité, météo, fériés) sont imposés par le générateur.

**7. Comment avez-vous nettoyé, annoté et séparé les données sans fuite ?**

Preuve à présenter : Pipeline, exemples avant/après, splits et règles anti-doublons.

* Réponse du groupe :**Nettoyage** : « N/A » des zones 264/265 lu comme texte et non comme valeur manquante ; zones 264 (Unknown) et 265 (Outside of NYC) exclues des KPI (pas de géométrie) ; géométries regroupées par identifiant de zone ; classification explicite des zones (résidentiel, centre d’affaires, parc ou spécial, aéroport) à partir d’une liste d’identifiants, après qu’un premier essai par mot-clé ait classé à tort « Park Slope » ou « Kips Bay » en parcs ; valeurs manquantes des capteurs (\~0,3 %) interpolées par zone sur 3 jours maximum (881 valeurs pour les déchets).
* **Anti-doublons** : clé primaire `(date, zone\\\_id)` en base.
* **Splits chronologiques** (`dim\\\_date.split`), sans mélange aléatoire : train 2023-01-01 → 2025-02-28, validation 2025-03-01 → 2025-07-31, test 2025-08-01 → 2025-12-31. Modèle entraîné sur le train seul ; variables retardées d’au moins 7 jours (horizon 7 jours).
* **Point à corriger** : l’interpolation utilise les valeurs voisines, y compris de part et d’autre de la frontière entre splits. Fuite négligeable (≤ 3 jours) mais à supprimer en interpolant séparément par split.

**8. Quelle insuffisance des données risque de fausser votre conclusion ?**

Preuve à présenter : Biais identifié et mesure concrète pour en réduire l’effet.

Réponse du groupe : Preuve à présenter : Biais identifié et mesure concrète pour en réduire l’effet.

Réponse du groupe : Les données étant générées avec leurs propres règles, un modèle qui retrouve ces règles paraît meilleur qu’il ne le serait en réel : les MAE actuels valident le pipeline, pas la performance réelle. Mesures prévues **\[Prévu]** : comparer avec des données ouvertes réelles si la licence et la fraîcheur le permettent (collecte des déchets, consommation d’eau, demandes 311, météo NOAA) et tester la robustesse avec des ruptures non modélisées. Non encore réalisé.

**9. Quels composants fonctionnent et comment échangent-ils les données ?**

Preuve à présenter : Schéma d’architecture à jour et exemple de requête API.

Réponse du groupe : \[à compléter]

**10. Pourquoi avez-vous choisi ces frameworks plutôt qu’une solution plus simple ?**

Preuve à présenter : Justification liée au besoin, aux compétences et aux ressources.

Réponse du groupe : \[à compléter]

**11. Comment gérez-vous l’état, la mémoire, les erreurs et les reprises ?**

Preuve à présenter : Modèle d’état, journal d’une exécution et exemple d’échec traité.

Réponse du groupe : \[à compléter]

Correction prioritaire sur les données ou la conception : \[à compléter]

## Fonctionnement des agents et protocole

**12. Quelle décision l’agent prend-il et quels outils appelle-t-il ?**

Preuve à présenter : Trace d’un scénario avec entrées, décisions, appels et sorties.

Réponse du groupe : \[à compléter]

**13. Quel résultat obtenez-vous avec des règles ou une chaîne sans agent ?**

Preuve à présenter : Comparaison sur les mêmes cas, avec le même budget de calcul.

Réponse du groupe : \[à compléter]

**14. Si plusieurs agents collaborent, quel rôle distinct remplit chacun ?**

Preuve à présenter : Rôles, échanges et comparaison avec un agent unique. Sinon expliquer l’absence de multi-agents.

Réponse du groupe : \[à compléter]

**15. Comment vérifiez-vous une sortie du LLM avant de l’utiliser ?**

Preuve à présenter : Validation du format, contrôle des sources, test métier et cas d’abstention.

Réponse du groupe : \[à compléter]

**16. Quelles actions exigent une validation humaine et quelles limites imposez-vous ?**

Preuve à présenter : Permissions, plafond d’appels, coût maximal et scénario d’action refusée.

Réponse du groupe : \[à compléter]

**17. Quels tests prouvent que les fonctions annoncées marchent réellement ?**

Preuve à présenter : Tests automatisés, résultat daté et anomalies encore ouvertes.

Réponse du groupe : \[à compléter]

**18. Quels indicateurs avez-vous choisis et pourquoi conviennent-ils au projet ?**

Preuve à présenter : Définition des métriques, baseline et protocole de mesure.

Réponse du groupe : \[à compléter]

Correction prioritaire sur l’agent ou les tests : \[à compléter]

## Résultats et déploiement

**19. Quels résultats chiffrés obtenez-vous sur un jeu de test distinct ?**

Preuve à présenter : Tableau baseline/solution, taille du test, latence et coût par scénario.

Réponse du groupe : \[à compléter]

**20. Quels sont vos trois échecs les plus importants et comment les expliquez-vous ?**

Preuve à présenter : Entrées, sorties observées, causes probables et correction envisagée.

Réponse du groupe : \[à compléter]

**21. Quelles entrées difficiles ou hostiles avez-vous testées ?**

Preuve à présenter : Données manquantes, outil indisponible, prompt malveillant, réponse non sourcée.

Réponse du groupe : \[à compléter]

**22. Un autre groupe peut-il installer et lancer votre application ?**

Preuve à présenter : README vérifié, dépendances fixées et commande de lancement.

Réponse du groupe : \[à compléter]

**23. Quelle version est déployée et quel parcours avez-vous validé ?**

Preuve à présenter : Commit ou version, environnement, démonstration et résultat du test de bout en bout.

Réponse du groupe : \[à compléter]

**24. Comment protégez-vous les secrets, les accès et les données ?**

Preuve à présenter : Configuration sans secrets, contrôle d’accès et politique de journalisation.

Réponse du groupe : \[à compléter]

### Résultats à comparer

|Indicateur|Baseline|Solution|Jeu ou scénario|
|-|-|-|-|
|\[métrique métier]|\[valeur]|\[valeur]|\[référence]|
|Latence et coût|\[valeur]|\[valeur]|\[référence]|

Pour un projet industriel, démontrer en simulation et expliquer les limites du passage au réel. Pour un projet informatique, montrer les droits accordés aux outils et les actions qui exigent une validation humaine.

## Contributions et corrections

**25. Quelle contribution chaque membre peut-il présenter et expliquer ?**

Preuve à présenter : Tâches, commits, résultats et revue croisée. Chaque membre explique au moins un choix technique.

Réponse du groupe : \[à compléter]

**26. Quel blocage menace le prochain jalon et de quelle aide avez-vous besoin ?**

Preuve à présenter : Cause précise, tentatives déjà faites et décision attendue.

Réponse du groupe : \[à compléter]

**27. Quelle affirmation ou fonctionnalité devez-vous corriger après cette revue ?**

Preuve à présenter : Écart entre prévu et observé, cause et preuve de correction attendue.

Réponse du groupe : \[à compléter]

**28. Quelles sont vos trois prochaines actions prioritaires ?**

Preuve à présenter : Action observable, responsable nommé, échéance et test d’acceptation.

Réponse du groupe : \[à compléter]

**29. Faut-il réduire le périmètre ou revoir le choix des données ou des outils ?**

Preuve à présenter : Décision argumentée et effet sur le planning et les livrables.

Réponse du groupe : \[à compléter]

**30. Quelle preuve présenterez-vous lors du prochain point d’avancement ?**

Preuve à présenter : Jalon vérifiable et scénario exact qui permettra de le valider.

Réponse du groupe : \[à compléter]

### Plan de correction à valider

|Action prioritaire|Responsable|Échéance|Preuve de correction|
|-|-|-|-|
|1. \[action]|\[nom]|\[date]|\[test ou résultat attendu]|
|2. \[action]|\[nom]|\[date]|\[test ou résultat attendu]|
|3. \[action]|\[nom]|\[date]|\[test ou résultat attendu]|

Décision de la revue : \[poursuivre / corriger avant le prochain jalon / réduire le périmètre]

Commentaire de l’enseignant : \[à compléter]

