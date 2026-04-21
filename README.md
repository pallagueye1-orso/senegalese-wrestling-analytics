# 🥋 Analyse Économique de la Lutte Sénégalaise (Lamb) — PySpark

> Projet Master 1 BIDABI — Université Paris 13  
> Cours : Traitement de données massives avec PySpark  
> Groupe : Palla · Mouhammed · Havar

---

## 📌 Contexte

La lutte sénégalaise (*lamb*) constitue à la fois un sport populaire et un secteur économique structuré au Sénégal. Ce projet analyse un dataset synthétique de combats afin de comprendre les dynamiques économiques du lamb : revenus générés, performances des lutteurs, évolution du chiffre d'affaires dans le temps.

L'ensemble du pipeline est implémenté avec **Apache PySpark**, exécuté sur **Google Colab**.

---

## 👥 Répartition du travail

| Partie | Responsable | Contenu |
|---|---|---|
| **Partie 1 — Data Engineering** | Palla | Chargement, schéma explicite, nettoyage, création de variables |
| **Partie 2 — Analyse exploratoire & Visualisation** | Mouhammed | EDA, graphiques, KPIs économiques |
| **Partie 3 — Transformations avancées, Optimisation & ML** | Havar | Jointures, window functions, cache, modèle ML |

---

## 📁 Structure du dépôt

```
.
├── README.md
├── data/
│   ├── lamb.csv                          # Dataset brut (synthétique)
│   └── lamb_clean.parquet/               # Dataset nettoyé (généré par Partie 1)
├── notebooks/
│   ├── Partie1_DataEngineering_Palla.ipynb
│   ├── Partie2_EDA_Mouhammed.ipynb
│   └── Partie3_Avancee_Havar.ipynb
└── slides/
    └── presentation.pdf                  # Slides de soutenance
```

---

## 📊 Dataset — `lamb.csv`

| Colonne | Type | Description |
|---|---|---|
| `fight_id` | Entier | Identifiant unique du combat |
| `date` | Date | Date du combat (YYYY-MM-DD) |
| `wrestler_1` | Entier | ID du premier lutteur |
| `wrestler_2` | Entier | ID du second lutteur |
| `winner` | Entier nullable | ID du vainqueur (`null` = combat nul/annulé) |
| `revenue` | Flottant nullable | Recettes billetterie en FCFA |
| `tickets_sold` | Entier | Nombre de billets vendus |
| `sponsor_revenue` | Entier | Revenus sponsors en FCFA |
| `arena` | Texte | Arène du combat |
| `promoter` | Texte | Promoteur du combat |

**Dimensions :** 2 000 lignes × 10 colonnes  
**Période couverte :** Janvier 2022 → Juin 2024

---

## 🔧 Partie 1 — Data Engineering (Palla)

### Ce qui est fait

- Schéma PySpark explicite avec justification de chaque type
- Audit qualité complet (nulls, doublons, types corrompus, incohérences métier)
- Gestion des valeurs manquantes colonne par colonne (stratégie documentée)
- Détection et correction de la valeur sentinel `-1000` dans `revenue`
- Déduplication sur `fight_id`
- Conversion `date` → `DateType`, `winner` → `IntegerType`
- Création des variables dérivées : `total_revenue`, `revenue_per_ticket`, `winner_label`, `has_ticket_revenue`, `year`, `month`
- Export du dataset propre en **Parquet** (optimisé pour PySpark) et CSV

### Variables créées

| Variable | Calcul | Utilité |
|---|---|---|
| `total_revenue` | `coalesce(revenue, 0) + sponsor_revenue` | KPI principal — CA par combat |
| `revenue_per_ticket` | `revenue / tickets_sold` | Valeur unitaire d'un combat |
| `winner_label` | ID vainqueur ou `"Nul/Annulé"` | Lisibilité des analyses |
| `has_ticket_revenue` | `revenue IS NOT NULL` | Filtre qualité billetterie |
| `year` / `month` | Extraits de `date` | Analyses temporelles |

### ⚠️ Hypothèse importante pour les Parties 2 & 3

Le `total_revenue` des combats sans données de billetterie (`has_ticket_revenue = false`) ne contient **que les revenus sponsors**. Pour des analyses de CA absolu précises, filtrer sur `has_ticket_revenue = true` ou mentionner cette hypothèse explicitement.

---

## ▶️ Comment exécuter

### Prérequis

- Compte Google (pour Colab)
- Fichier `lamb.csv` uploadé sur votre Google Drive

### Lancement

1. Ouvrir le notebook souhaité dans Google Colab
2. Adapter `FILE_PATH` au chemin réel du fichier sur votre Drive :
   ```python
   FILE_PATH = "/content/drive/MyDrive/lamb.csv"
   ```
3. Exécuter toutes les cellules dans l'ordre (`Runtime > Run all`)

> **Important :** exécuter la Partie 1 en premier — elle génère `lamb_clean.parquet` utilisé par les Parties 2 et 3.

### Dépendances

PySpark est installé automatiquement dans chaque notebook :
```bash
!pip install pyspark --quiet
```
Aucune autre installation n'est requise.

---

## 📋 Choix techniques principaux

| Décision | Justification |
|---|---|
| Schéma explicite vs `inferSchema` | Évite une double lecture du fichier, garantit la cohérence des types |
| `spark.sql.shuffle.partitions = 4` | Valeur par défaut (200) excessive pour 2 000 lignes → gains de performance |
| Export Parquet | Préserve les types, lecture 2-10x plus rapide qu'un CSV |
| Sentinel -1000 → null | Valeur codée impossible métier ; laisser -1000 biaiserait toutes les agrégations |
| `coalesce(revenue, 0)` dans `total_revenue` | Permet d'avoir un KPI calculé sur toutes les lignes ; compensé par le flag `has_ticket_revenue` |

---

## 👨‍🏫 Contexte académique

Ce projet répond aux consignes suivantes :

- [x] Ingestion et préparation (schéma explicite, nettoyage, documentation)
- [x] Analyse exploratoire (CA, top lutteurs, évolution temporelle)
- [x] Transformations avancées (jointures, agrégations, window functions)
- [x] Optimisation (cache, partitionnement, explain plan)
- [ ] Machine Learning (bonus)
- [x] Visualisation (≥ 3 graphiques)

---

*Dataset synthétique construit pour reproduire de manière réaliste l'environnement économique de la lutte sénégalaise.*
