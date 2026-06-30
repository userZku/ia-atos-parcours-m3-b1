# Note d'identification des sources — Acerox Métallurgie

> Document remis à Sébastien Marchand (chef de projet industrialisation Acerox).

> Public visé : décideur métier non technique.

> Auteur : Théo / Copilot - Date : 30/06/2026

## 1. Contexte

Acerox Métallurgie exploite un modèle existant de prédiction des défauts qualité en production. Dans le cadre de la mission FastIA, l'objectif est d'identifier les sources de données les plus utiles pour enrichir ce modèle, sans engager à ce stade un chantier technique complet. Trois sources ont été fournies (capteurs IoT, export ERP, logs machines). Cette note vise à donner une vision claire de leur valeur métier, de leur qualité apparente et des points de vigilance RGPD avant de lancer l'implémentation d'ingestion.

## 2. Demande métier reformulée

Ce que Sébastien a demandé : enrichir le modèle existant avec de nouvelles données pour mieux traiter les défauts qualité.

Ce que je comprends qu'il cherche vraiment : améliorer la prise de décision opérationnelle (maintenance, supervision de ligne, priorisation des actions correctives) en détectant plus tôt les situations à risque de non-conformité, avec un focus probable sur les lignes et sites les plus sensibles.

## 3. Inventaire des sources

| Source | Format | Volume | Fréquence | Qualité observée | Risques RGPD | Pertinence métier |
|---|---|---|---|---|---|---|
| capteurs_iot.csv | CSV | 51 000 lignes, 7 colonnes, ~10,59 Mo en RAM | Continue (mesures horodatées sur la période fournie) | Qualité globalement bonne ; 749 valeurs manquantes sur vibration_mms (1,47%) ; 1 000 doublons exacts | Faible à moyen : pas de donnée personnelle directe, mais identifiants techniques (sensor_id, line_id) recoupables avec d'autres sources | Haute : source la plus directement liée aux conditions machine et aux défauts |
| erp_export.json | JSON | 2 000 lignes, 9 colonnes, ~0,74 Mo en RAM | Batch (rythme exact à confirmer, probablement quotidien) | Structure cohérente ; 109 valeurs manquantes sur ouvrier_id (5,45%) | Moyen : ouvrier_id est pseudonymisé mais potentiellement ré-identifiable par croisement (planning RH, logs, horaires) | Haute : apporte le contexte ordre/statut/quantité nécessaire à l'analyse métier |
| logs_machines.log | Texte semi-structuré | 30 000 lignes, 1,83 Mo fichier (~3,20 Mo en RAM après chargement) | Continue | 0 ligne vide, 1 doublon ; répartition INFO/WARN/ERROR exploitable ; parsing plus poussé à prévoir ensuite | Faible à moyen : pas d'email/téléphone/IP détectés, mais présence de marqueurs d'identifiants (user/employee/id/token) dans 425 lignes | Moyenne : utile pour corroborer les incidents, mais valeur dépendante du parsing |
| rapports_techniques_2024/*.md | Markdown non structuré | 5 rapports, 45 lignes, ~2,8 Ko | Ponctuelle / événementielle (production de rapports après incident ou dérive) | Corpus court, homogène et lisible ; métadonnées déjà présentes dans l'en-tête (référence, site, ligne, date) ; nécessite segmentation pour recherche sémantique | Moyen : présence d'identifiants de personnel dans le texte libre (ex. responsable `EMP-2424`) et risque de recoupement avec ERP/logs | Moyenne : utile pour l'aide au diagnostic humain et la capitalisation des incidents, moins prioritaire pour la prédiction tabulaire |

## 4. Recommandations

- Prioriser l'ingestion de capteurs_iot.csv et erp_export.json : c'est le meilleur ratio valeur métier / effort immédiat.
- Intégrer logs_machines.log en phase 2 : utile, mais la valeur dépend d'un travail de parsing et de normalisation qui n'est pas nécessaire pour démarrer.
- Positionner les rapports techniques comme source complémentaire de connaissance : pertinents pour un moteur de recherche sémantique ou une aide au diagnostic, mais pas prioritaires pour enrichir directement le modèle tabulaire actuel.
- Traiter les doublons IoT avant usage modèle (1 000 lignes) et monitorer la qualité du champ vibration_mms (1,47% manquants).
- Encadrer strictement l'usage de ouvrier_id : ne le conserver que si la finalité métier le justifie ; sinon pseudonymisation renforcée (hachage) et minimisation des accès.
- Mettre en place un contrôle qualité simple à l'ingestion (manquants, doublons, format de date, distribution des statuts) pour éviter la dégradation silencieuse des données.

## 5. Points à clarifier avec Sébastien

1. Quel est le rythme réel de mise à jour de l'export ERP (journalier, pluri-journalier, hebdomadaire) et son délai de disponibilité ?
2. Quel KPI prioritaire doit être optimisé en premier (taux de non-conformité, rebuts, arrêts, délai de détection) ?
3. Le champ ouvrier_id est-il réellement nécessaire à la prédiction ou seulement utile pour l'explication opérationnelle ?
4. Existe-t-il un dictionnaire de données officiel (définitions statut, line_id, règles de saisie ERP) ?
5. Le format des logs machines est-il stable dans le temps ou susceptible de changer selon version système/atelier ?
6. Les rapports techniques sont-ils produits de manière systématique après chaque incident, ou seulement sur certains cas jugés critiques ?
7. Comment une non-conformité est-elle définie et validée chez Acerox (seuils, contrôle qualité, source de vérité), et à quel moment ce label est-il disponible dans les données ?

## 6. Limites de cette note

- Cette note couvre l'identification et la priorisation des sources, pas une EDA statistique approfondie.
- Aucune AIPD juridique formelle n'a été réalisée ; une validation DPO reste nécessaire avant industrialisation multi-sources.
- Aucun test de performance modèle n'a été conduit avec ces nouvelles sources ; l'impact réel devra être mesuré en phase d'intégration.
- La fréquence exacte de certaines sources (notamment ERP) reste à confirmer avec les équipes Acerox.

---

Note produite par Théo, 30/06/2026, dans le cadre du brief M3-B1 ATOS.
