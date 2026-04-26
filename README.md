# 🏥 NF26 - Solution Décisionnelle pour un Hôpital

Ce dépôt regroupe l'ensemble des scripts réalisés dans le cadre de l'UV **NF26** à l'UTC, dédiée aux **bases de données et à l'informatique décisionnelle**.

Le projet consiste à construire un **pipeline ETL complet** à partir de données fictives d'un établissement hospitalier, depuis l'ingestion de fichiers plats jusqu'à la visualisation dans **Power BI**, en passant par plusieurs couches de transformation sur **Teradata**.

<br/>

## 📌 Vue d'ensemble

| Couche | Rôle | Technologie |
|--------|------|-------------|
| **Source** | Fichiers plats journaliers (`.txt`, délimiteur `;`) | Système de fichiers |
| **STG** *(Staging)* | Ingestion brute des données sources | Teradata + TPT |
| **WRK** *(Work)* | Nettoyage, transformation et enrichissement | Teradata SQL (BTEQ) |
| **SOC** *(Socle)* | Données certifiées et consolidées | Teradata SQL (BTEQ) |
| **TCH** *(Technique)* | Suivi des exécutions et traçabilité | Teradata SQL (BTEQ) |
| **Reporting** | Tableaux de bord décisionnels | Power BI |

<br/>

## 🗂 Architecture du pipeline ETL

```
DATA/BDD_HOSPITAL_YYYYMMDD/
    ├── CHAMBRE.txt
    ├── CONSULTATION.txt
    ├── HOSPITALISATION.txt
    ├── MEDICAMENT.txt
    ├── PATIENT.txt
    ├── PERSONNEL.txt
    └── TRAITEMENT.txt
          │
          ▼  TPT (Teradata Parallel Transporter)
       STG (Staging)
          │
          ▼  insert_to_wrk_from_stg.sql
       WRK (Work)
          │
          ▼  insert_to_soc_from_wrk.sql
       SOC (Socle certifié)
          │
          ▼  EXPORT/
       Power BI (Reporting)
```

Le suivi de chaque exécution est assuré par la couche **TCH** (`T_SUIV_RUN`, `T_SUIV_TRMT`), alimentée tout au long du pipeline.

<br/>

## 📁 Détail des scripts

### Initialisation

| Fichier | Contenu |
|---------|---------|
| [`install_SID.sh`](SCRIPTS/install_SID.sh) | Installation de l'environnement Teradata |
| [`create_db.sql`](SCRIPTS/create_db.sql) | Création des bases de données STG, WRK, SOC, TCH |
| [`create_table_stg.sql`](SCRIPTS/create_table_stg.sql) | Création des tables de la couche Staging |
| [`create_table_wrk.sql`](SCRIPTS/create_table_wrk.sql) | Création des tables de la couche Work |
| [`create_table_soc.sql`](SCRIPTS/create_table_soc.sql) | Création des tables de la couche Socle |
| [`create_table_tch.sql`](SCRIPTS/create_table_tch.sql) | Création des tables de suivi technique |

---

### Chargement STG - Ingestion via TPT

| Fichier | Table cible |
|---------|-------------|
| [`load_chambre.tpt`](SCRIPTS/load_chambre.tpt) | `STG.CHAMBRE` |
| [`load_consultation.tpt`](SCRIPTS/load_consultation.tpt) | `STG.CONSULTATION` |
| [`load_hospitalisation.tpt`](SCRIPTS/load_hospitalisation.tpt) | `STG.HOSPITALISATION` |
| [`load_medicament.tpt`](SCRIPTS/load_medicament.tpt) | `STG.MEDICAMENT` |
| [`load_patient.tpt`](SCRIPTS/load_patient.tpt) | `STG.PATIENT` |
| [`load_personnel.tpt`](SCRIPTS/load_personnel.tpt) | `STG.PERSONNEL` |
| [`load_traitement.tpt`](SCRIPTS/load_traitement.tpt) | `STG.TRAITEMENT` |

---

### Transformation & Chargement

| Fichier | Contenu |
|---------|---------|
| [`insert_to_wrk_from_stg.sql`](SCRIPTS/insert_to_wrk_from_stg.sql) | Transformation et chargement STG → WRK |
| [`insert_to_soc_from_wrk.sql`](SCRIPTS/insert_to_soc_from_wrk.sql) | Chargement certifié WRK → SOC |
| [`delete_table_stg.sql`](SCRIPTS/delete_table_stg.sql) | Purge des tables STG après traitement |
| [`delete_table_wrk.sql`](SCRIPTS/delete_table_wrk.sql) | Purge des tables WRK après traitement |

---

### Suivi des exécutions (TCH)

| Fichier | Contenu |
|---------|---------|
| [`create_exec_run_id.sql`](SCRIPTS/create_exec_run_id.sql) | Initialisation d'un identifiant de run |
| [`update_exec_id_ok.sql`](SCRIPTS/update_exec_id_ok.sql) | Marquage d'une exécution en succès |
| [`update_exec_id_ko.sql`](SCRIPTS/update_exec_id_ko.sql) | Marquage d'une exécution en échec |
| [`update_run_id_ok.sql`](SCRIPTS/update_run_id_ok.sql) | Clôture d'un run en succès |
| [`update_run_id_ko.sql`](SCRIPTS/update_run_id_ko.sql) | Clôture d'un run en échec |

---

### Orchestration

| Fichier | Contenu |
|---------|---------|
| [`LAUNCH_LOAD_SID.sh`](SCRIPTS/LAUNCH_LOAD_SID.sh) | Script principal - orchestre l'ensemble du pipeline ETL pour chaque dossier journalier |
| [`run_tpt.sh`](SCRIPTS/run_tpt.sh) | Lance les scripts TPT pour un répertoire source donné |

<br/>

## 🗃 Modèle de données

Le diagramme UML complet des couches STG, WRK et SOC est disponible dans [`UML/UML.plantUML`](UML/UML.plantUML).

### Entités principales

| Entité | Description |
|--------|-------------|
| `PATIENT` / `O_INDIV` | Données démographiques des patients |
| `PERSONNEL` / `O_STFF` | Personnel médical et administratif |
| `CHAMBRE` / `R_ROOM` | Chambres de l'établissement |
| `MEDICAMENT` / `R_MEDC` | Référentiel médicaments |
| `CONSULTATION` | Consultations médicales |
| `HOSPITALISATION` | Séjours d'hospitalisation |
| `TRAITEMENT` | Traitements prescrits lors des consultations |

<br/>

## 🚀 Exécution

### Prérequis

- Teradata (instance locale ou distante) avec BTEQ et TPT installés
- Bash (Linux ou WSL sous Windows)
- Power BI Desktop pour la visualisation

### Lancer le pipeline complet

```bash
# 1. Initialiser les bases de données et les tables
bteq < SCRIPTS/create_db.sql
bteq < SCRIPTS/create_table_stg.sql
bteq < SCRIPTS/create_table_wrk.sql
bteq < SCRIPTS/create_table_soc.sql
bteq < SCRIPTS/create_table_tch.sql

# 2. Lancer le pipeline ETL (traite tous les dossiers BDD_HOSPITAL_*)
cd SCRIPTS
bash LAUNCH_LOAD_SID.sh
```

Le script `LAUNCH_LOAD_SID.sh` traite automatiquement jusqu'à 10 dossiers journaliers et écrit un fichier de log `LAUNCH_LOAD_SID.log` à chaque exécution.

<br/>

## 🧰 Technologies utilisées

- **Teradata** - base de données relationnelle, SQL BTEQ
- **TPT** *(Teradata Parallel Transporter)* - chargement massif de fichiers plats
- **Bash** - orchestration et automatisation du pipeline
- **Power BI** - visualisation et tableaux de bord décisionnels
- **PlantUML** - modélisation UML du schéma de données

<br/>

## 📄 Licence

Ce projet est distribué sous licence **MIT** - voir le fichier [LICENSE](LICENSE) pour plus d'informations.

<br/>

## 👤 Auteurs

- **Alexandre L.**
- **[Martin C.](https://github.com/martincrz)**
- **Nassim S.**
- **[Tobias S.](https://github.com/tobiasinfo)**
- **[Sacha Sz](https://github.com/sacha-sz)**

<br/>

## 🔗 Références

- [🔒 Cours NF26 sur Moodle (accès UTC requis)](https://moodle.utc.fr/)
- [UTC - Université de Technologie de Compiègne](https://www.utc.fr/)
- [Teradata Documentation](https://docs.teradata.com/)
- [PlantUML](https://plantuml.com/)