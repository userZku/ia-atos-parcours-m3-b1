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
    %% Sources identifiees en M3-B1
    SRC_IOT[📡 capteurs_iot.csv<br/>CSV<br/>51 000 lignes, 7 colonnes<br/>Frequence continue]
    SRC_ERP[📋 erp_export.json<br/>JSON<br/>2 000 lignes, 9 colonnes<br/>Frequence batch a confirmer]
    SRC_LOG[📝 logs_machines.log<br/>Texte semi-structure<br/>30 000 lignes<br/>Frequence continue]
    SRC_PDF[📄 rapports_maintenance.pdf<br/>Bonus potentiel<br/>non integre en M3-B1]

    INGEST[🔄 Ingestion<br/>à concevoir en M3-B2]
    BDD[(🗄️ BDD pivot<br/>SQLite)]
    MODEL[🧠 Modèle existant Acerox<br/>prédiction défauts qualité]

    SRC_IOT -->|mesures machine| INGEST
    SRC_ERP -->|ordres + statut| INGEST
    SRC_LOG -->|events INFO/WARN/ERROR| INGEST
    SRC_PDF -.->|extraction OCR/NLP a etudier| INGEST
    INGEST -->|normalisation + dédup| BDD
    BDD -->|consommée par| MODEL

    classDef source fill:#e1f5ff,stroke:#0277bd
    classDef bonus fill:#f5f5f5,stroke:#6b7280,stroke-dasharray: 4 4
    classDef tofix fill:#fff4e1,stroke:#c97a00,stroke-dasharray: 5 5
    class SRC_IOT,SRC_ERP,SRC_LOG source
    class SRC_PDF bonus
    class INGEST tofix
```

## Légende

> Reformule en 5 lignes max ce que le schéma raconte (qui produit quelle
> donnée, qui consomme, contraintes critiques).

- **Producteurs** : capteurs atelier (IoT), systeme ERP, systeme de logs machines.
- **Consommateur final** : modele existant Acerox de prediction des defauts qualite, alimente via la BDD pivot.
- **Contraintes frequence** : IoT et logs en continu, ERP en batch (cadence exacte a confirmer).
- **Contraintes qualite** : doublons IoT, valeurs manquantes sur `vibration_mms` et `ouvrier_id`, logs a parser proprement.
- **Contraintes RGPD** : risque indirect de re-identification via `ouvrier_id` en croisement multisources.

## Décisions associées

- Source(s) retenues en priorité : `capteurs_iot.csv` et `erp_export.json`.
- Source(s) écartées : aucune source completement ecartee; `logs_machines.log` est reportee en phase 2.
- Source bonus (PDF) traitée ? non, car hors perimetre M3-B1 et necessite une extraction textuelle specifique.

---

*Schéma produit par Theo, 30/06/2026, dans le cadre du brief M3-B1 ATOS.*
