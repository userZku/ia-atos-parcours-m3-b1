# Schéma des flux de données — Acerox Métallurgie

> Schéma Mermaid à compléter. Doit montrer :
> - **Sources** (capteurs IoT, ERP, logs, *bonus PDF*)
> - **Ingestion** (à concevoir en M3-B2)
> - **BDD pivot** (à modéliser en M3-B2)
> - **Modèle existant** Acerox (placeholder, hors-sujet ici)
>
> Légende explicite : qui produit, qui consomme, contraintes.

## Schéma

```mermaid
flowchart LR
    %% Sources identifiées en M3-B1
    SRC_IOT[📡 capteurs_iot.csv<br/>CSV<br/>51 000 lignes, 7 colonnes<br/>Fréquence continue]
    SRC_ERP[📋 erp_export.json<br/>JSON<br/>2 000 lignes, 9 colonnes<br/>Fréquence batch à confirmer]
    SRC_LOG[📝 logs_machines.log<br/>Texte semi-structuré<br/>30 000 lignes<br/>Fréquence continue]
    SRC_RPT[📄 rapports_techniques_2024<br/>Markdown non structuré<br/>5 rapports, 45 lignes<br/>Fréquence événementielle]

    INGEST[🔄 Ingestion<br/>à concevoir en M3-B2]
    BDD[(🗄️ BDD pivot<br/>SQLite)]
    MODEL[🧠 Modèle existant Acerox<br/>prédiction défauts qualité]

    SRC_IOT -->|mesures machine| INGEST
    SRC_ERP -->|ordres + statut| INGEST
    SRC_LOG -->|events INFO/WARN/ERROR| INGEST
    SRC_RPT -.->|segmentation + recherche sémantique| INGEST
    INGEST -->|normalisation + dédup| BDD
    BDD -->|consommée par| MODEL

    classDef source fill:#e1f5ff,stroke:#0277bd
    classDef bonus fill:#f5f5f5,stroke:#6b7280,stroke-dasharray: 4 4
    classDef tofix fill:#fff4e1,stroke:#c97a00,stroke-dasharray: 5 5
    class SRC_IOT,SRC_ERP,SRC_LOG source
    class SRC_RPT bonus
    class INGEST tofix
```

## Légende

> Reformule en 5 lignes max ce que le schéma raconte (qui produit quelle
> donnée, qui consomme, contraintes critiques).

- **Producteurs** : capteurs atelier (IoT), système ERP, système de logs machines et rapports techniques rédigés après incident/intervention.
- **Consommateur final** : modèle existant Acerox de prédiction des défauts qualité, alimenté via la BDD pivot.
- **Contraintes fréquence** : IoT et logs en continu, ERP en batch (cadence exacte à confirmer).
- **Contraintes qualité** : doublons IoT, valeurs manquantes sur `vibration_mms` et `ouvrier_id`, logs à parser proprement, rapports à segmenter avant exploitation.
- **Contraintes RGPD** : risque indirect de ré-identification via `ouvrier_id` et présence possible d'identifiants opérateur dans les rapports techniques.

## Décisions associées

- Source(s) retenues en priorité : `capteurs_iot.csv` et `erp_export.json`.
- Source(s) écartées : aucune source complètement écartée ; `logs_machines.log` est reportée en phase 2.
- Source bonus / complémentaire traitée ? oui : `rapports_techniques_2024`, à valoriser surtout pour la recherche sémantique et l'aide au diagnostic.

---

*Schéma produit par Théo, 30/06/2026, dans le cadre du brief M3-B1 ATOS.*
