# Notes d'entretien — Sébastien Marchand (Acerox)

> Notes prises au fil de l'eau pendant l'entretien fictif 9h30-10h00.
> **5 catégories à pré-remplir** avant l'entretien. Tu complètes au fil
> de l'eau. Distinction explicite *dit* (ce que Sébastien a effectivement
> formulé) vs *interprété* (ta lecture).
>
> Conservé en livrable du brief.

**Date** : 30/06/2026 — **Durée** : 30 min — **Présents** : Sébastien Marchand (chef de projet industrialisation Acerox) + Théo (FastIA)

---

## 1. Besoin métier

> Quelle décision Sébastien veut-il améliorer ? Quel KPI métier ?

**Dit** : "On veut réduire le nombre de défauts. Il vaut mieux détecter trop de défauts que d'en laisser passer."

**Interprété** : priorité à la réduction des faux négatifs (défauts non détectés), même si cela augmente temporairement les alertes à vérifier en production. Le besoin porte sur une meilleure anticipation des non-conformités pour agir plus tôt (maintenance/réglage de ligne).

## 2. Sources et formats

> Qu'a-t-il à disposition ? Où ? Sous quel format ?

**Dit** : "On a des valeurs capteurs, des logs machines, et des données ERP."

**Interprété** : trois sources principales sont mobilisables à court terme : `capteurs_iot.csv` (mesures process), `erp_export.json` (ordres/statuts), `logs_machines.log` (événements terrain). Le modèle actuel exploite déjà une partie de l'existant, mais le périmètre exact des variables utilisées reste à confirmer.

## 3. Volumétrie et fréquence

> Combien de données ? À quelle cadence arrivent-elles ?

**Dit** : "Les données arrivent en continu."

**Interprété** : flux quasi temps réel pour IoT et logs ; fonctionnement plutôt batch pour l'ERP (fréquence exacte non précisée pendant l'entretien). Volumétrie observée ensuite : ~51k lignes IoT, ~2k lignes ERP, ~30k lignes logs sur la période fournie.

## 4. Contraintes (RGPD, sécurité, propriété)

> Quelles obligations légales ? Qui possède les données ? Quels accès
> sécurisés ?

**Dit** : "On préfère que ça reste local."

**Interprété** : forte attente de maîtrise interne des données (hébergement/accès). Vigilance RGPD sur les identifiants indirects, notamment `ouvrier_id` dans l'ERP, qui peut devenir ré-identifiable par croisement (horaires, logs, planning).

## 5. Critères de succès

> Comment Sébastien saura-t-il qu'on a réussi ?

**Dit** : "Objectif : -20% en 3 mois."

**Interprété** : cible métier lisible à court terme, probablement sur les non-conformités/rebuts. Le KPI précis (défauts détectés, défauts clients, rebuts internes, coût qualité) n'a pas été explicité et doit être verrouillé avant l'implémentation.

---

## Questions restées sans réponse

> Notes-le honnêtement — c'est précieux pour la note d'identification.

- Quelle est la définition exacte du KPI "-20% en 3 mois" (type de défaut, périmètre site/ligne, unité de mesure) ?
- Quelle est la fréquence de mise à jour de l'ERP et son délai de disponibilité réel ?
- Quelles variables sont actuellement utilisées par le modèle existant, et avec quelles performances de référence ?
- Le champ `ouvrier_id` est-il nécessaire au modèle ou uniquement utile pour l'analyse opérationnelle ?
- Existe-t-il un dictionnaire de données valide pour harmoniser statuts, lignes et règles de saisie ?

## Mes impressions à chaud (10 min après)

> 1 paragraphe, à toi.

Sébastien maîtrise bien l'objectif métier (réduire les défauts rapidement), mais les détails techniques du modèle actuel sont encore flous. L'entretien a confirmé la valeur des 3 sources (IoT, ERP, logs), avec une attente forte sur des résultats concrets à court terme et une sensibilité sur la maîtrise locale des données. La prochaine étape est de clarifier les zones d'ombre (KPI exact, périmètre variables, contraintes RGPD opérationnelles) pour éviter un cadrage trop large.

---

*Notes d'entretien produites par Théo, 30/06/2026.*
