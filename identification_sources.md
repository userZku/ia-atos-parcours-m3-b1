# Note d'identification des sources — Acerox Metallurgie

> Document remis a Sebastien Marchand (chef de projet industrialisation Acerox).

> Public vise: decideur metier non technique.

> Auteur: Theo / Copilot - Date: 30/06/2026

## 1. Contexte

Acerox Metallurgie exploite un modele existant de prediction des defauts qualite en production. Dans le cadre de la mission FastIA, l'objectif est d'identifier les sources de donnees les plus utiles pour enrichir ce modele, sans engager a ce stade un chantier technique complet. Trois sources ont ete fournies (capteurs IoT, export ERP, logs machines). Cette note vise a donner une vision claire de leur valeur metier, de leur qualite apparente et des points de vigilance RGPD avant de lancer l'implementation d'ingestion.

## 2. Demande metier reformulee

Ce que Sebastien a demande: enrichir le modele existant avec de nouvelles donnees pour mieux traiter les defauts qualite.

Ce que je comprends qu'il cherche vraiment: ameliorer la prise de decision operationnelle (maintenance, supervision de ligne, priorisation des actions correctives) en detectant plus tot les situations a risque de non-conformite, avec un focus probable sur les lignes et sites les plus sensibles.

## 3. Inventaire des sources

| Source | Format | Volume | Frequence | Qualite observee | Risques RGPD | Pertinence metier |
|---|---|---|---|---|---|---|
| capteurs_iot.csv | CSV | 51 000 lignes, 7 colonnes, ~10,59 Mo en RAM | Continue (mesures horodatees sur la periode fournie) | Qualite globalement bonne; 749 valeurs manquantes sur vibration_mms (1,47%); 1 000 doublons exacts | Faible a moyen: pas de donnee personnelle directe, mais identifiants techniques (sensor_id, line_id) recoupables avec d'autres sources | Haute: source la plus directement liee aux conditions machine et aux defauts |
| erp_export.json | JSON | 2 000 lignes, 9 colonnes, ~0,74 Mo en RAM | Batch (rythme exact a confirmer, probablement quotidien) | Structure coherente; 109 valeurs manquantes sur ouvrier_id (5,45%) | Moyen: ouvrier_id est pseudonymise mais potentiellement re-identifiable par croisement (planning RH, logs, horaires) | Haute: apporte le contexte ordre/statut/quantite necessaire a l'analyse metier |
| logs_machines.log | Texte semi-structure | 30 000 lignes, 1,83 Mo fichier (~3,20 Mo en RAM apres chargement) | Continue | 0 ligne vide, 1 doublon; repartition INFO/WARN/ERROR exploitable; parsing plus pousse a prevoir ensuite | Faible a moyen: pas d'email/telephone/IP detectes, mais presence de marqueurs d'identifiants (user/employee/id/token) dans 425 lignes | Moyenne: utile pour corroborer les incidents, mais valeur dependante du parsing |

## 4. Recommandations

- Prioriser l'ingestion de capteurs_iot.csv et erp_export.json: c'est le meilleur ratio valeur metier / effort immediat.
- Integrer logs_machines.log en phase 2: utile, mais la valeur depend d'un travail de parsing et de normalisation qui n'est pas necessaire pour demarrer.
- Traiter les doublons IoT avant usage modele (1 000 lignes) et monitorer la qualite du champ vibration_mms (1,47% manquants).
- Encadrer strictement l'usage de ouvrier_id: ne le conserver que si la finalite metier le justifie; sinon pseudonymisation renforcee (hachage) et minimisation des acces.
- Mettre en place un controle qualite simple a l'ingestion (manquants, doublons, format de date, distribution des statuts) pour eviter la degradation silencieuse des donnees.

## 5. Points a clarifier avec Sebastien

1. Quel est le rythme reel de mise a jour de l'export ERP (journalier, pluri-journalier, hebdomadaire) et son delai de disponibilite?
2. Quel KPI prioritaire doit etre optimise en premier (taux de non-conformite, rebuts, arrets, delai de detection)?
3. Le champ ouvrier_id est-il reellement necessaire a la prediction ou seulement utile pour l'explication operationnelle?
4. Existe-t-il un dictionnaire de donnees officiel (definitions statut, line_id, regles de saisie ERP)?
5. Le format des logs machines est-il stable dans le temps ou susceptible de changer selon version systeme/atelier?

## 6. Limites de cette note

- Cette note couvre l'identification et la priorisation des sources, pas une EDA statistique approfondie.
- Aucune AIPD juridique formelle n'a ete realisee; une validation DPO reste necessaire avant industrialisation multi-sources.
- Aucun test de performance modele n'a ete conduit avec ces nouvelles sources; l'impact reel devra etre mesure en phase d'integration.
- La frequence exacte de certaines sources (notamment ERP) reste a confirmer avec les equipes Acerox.

---

Note produite par Theo, 30/06/2026, dans le cadre du brief M3-B1 ATOS.
