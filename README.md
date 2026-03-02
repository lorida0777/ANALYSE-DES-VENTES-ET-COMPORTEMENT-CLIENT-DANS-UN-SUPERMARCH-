# ANALYSE-DES-VENTES-ET-COMPORTEMENT-CLIENT-DANS-UN-SUPERMARCH-
📊 PROJET ETL - ANALYSE DES VENTES ET COMPORTEMENT CLIENT DANS UN SUPERMARCHÉ

🎯 Contexte et Objectifs
Objectif Général
Mettre en place un processus ETL complet et construire un entrepôt de données décisionnel permettant d'analyser :

Les ventes par catégorie, période, zone géographique
Le profil et la fidélité des clients
Les performances par produit

Objectifs spécifique:

✅ Identifier, nettoyer et transformer des données hétérogènes
✅ Construire un modèle en étoile pour un entrepôt de données
✅ Implémenter un pipeline ETL (extraction, transformation, chargement)
✅ Réaliser des requêtes OLAP pour l'analyse multidimensionnelle
✅ Appliquer des techniques de restitution via tableaux de bord


🏗️ Architecture Technique
Stack Technologique
## 🏗️ Stack Technologique

| Composant            | Technologie              | Justification                               |
|----------------------|--------------------------|---------------------------------------------|
| Base de données      | PostgreSQL               | SGBD relationnel robuste supportant l’OLAP  |
| Support ETL / OLAP   | Python 3.x + Pandas      | Manipulation et transformation flexible     |
| ORM                  | SQLAlchemy               | Abstraction BD et portabilité               |
| Visualisation        | Matplotlib + Seaborn     | Graphiques analytiques personnalisables     |
| Génération données   | Faker + NumPy            | Simulation de données réalistes             |
| Documentation        | Jupyter Notebook         | Documentation interactive et analyses       |
---
Architecture du Système
```
┌─────────────────────────────────────────────────────────┐
│                    SOURCES DE DONNÉES                   │
├──────────────┬──────────────┬──────────────┬────────────┤
│   Produits   │   Clients    │   Magasins   │   Temps    │
└──────┬───────┴──────┬───────┴──────┬───────┴─────┬──────┘
       │              │              │             │
       └──────────────┼──────────────┼─────────────┘
                      │
              ┌───────▼────────┐
              │   EXTRACTION   │
              │   (Extract)    │
              └───────┬────────┘
                      │
              ┌───────▼────────┐
              │ TRANSFORMATION │
              │  (Transform)   │
              │  - Nettoyage   │
              │  - Validation  │
              │  - Enrichis.   │
              └───────┬────────┘
                      │
              ┌───────▼────────┐
              │   CHARGEMENT   │
              │     (Load)     │
              └───────┬────────┘
                      │
       ┌──────────────▼───────────────┐
       │  DATA WAREHOUSE (PostgreSQL) │
       │      Schéma en Étoile        │
       ├──────────────────────────────┤
       │  • dim_temps                 │
       │  • dim_produit               │
       │  • dim_client                │
       │  • dim_magasin               │
       │  • fait_ventes (centrale)    │
       └──────────────┬───────────────┘
                      │
       ┌──────────────▼───────────────┐
       │      ANALYSES OLAP           │
       │   Requêtes SQL Complexes     │
       └──────────────┬───────────────┘
                      │
       ┌──────────────▼───────────────┐
       │     VISUALISATIONS           │
       │  Dashboards + Rapports       │
       └──────────────────────────────┘
```
---
📊 Modélisation des Données
Schéma en Étoile (Star Schema)
Le modèle dimensionnel adopté est un schéma en étoile avec :

1 table de faits (fait_ventes) au centre
4 tables de dimensions autour
```
sql
┌─────────────────────┐
│    DIM_TEMPS        │
├─────────────────────┤
│ date_id (PK)        │◄─────┐
│ date_complete       │      │
│ jour, mois, annee   │      │
│ trimestre           │      │
│ jour_semaine        │      │
│ est_weekend         │      │
│ est_ferie           │      │
└─────────────────────┘      │
                             │
┌─────────────────────┐      │
│   DIM_PRODUIT       │      │
├─────────────────────┤      │
│ produit_id (PK)     │◄─────┤
│ code_produit        │      │
│ nom_produit         │      │      ┌──────────────────┐
│ categorie           │      │      │   FAIT_VENTES    │
│ fournisseur         │      ├─────►│  (Table Faits)   │
│ prix_unitaire       │      │      ├──────────────────┤
│ stock_disponible    │      │      │ vente_id (PK)    │
└─────────────────────┘      │      │ date_id (FK)     │
                             │      │ produit_id (FK)  │
┌─────────────────────┐      │      │ client_id (FK)   │
│    DIM_CLIENT       │      │      │ magasin_id (FK)  │
├─────────────────────┤      │      │ quantite         │
│ client_id (PK)      │◄─────┤      │ montant_unitaire │
│ age, sexe           │      │      │ montant_total    │
│ tranche_age         │      │      │ remise           │
│ ville, region       │      │      │ montant_net      │
│ statut_fidelite     │      │      └──────────────────┘
│ segment_client      │      │
│ date_inscription    │      │
└─────────────────────┘      │
                             │
┌─────────────────────┐      │
│   DIM_MAGASIN       │      │
├─────────────────────┤      │
│ magasin_id (PK)     │◄─────┘
│ nom_magasin         │
│ ville, region       │
│ surface_m2          │
│ type_magasin        │
└─────────────────────┘
```


---

# 📊 Modélisation des Données

## ⭐ Schéma en Étoile (Star Schema)

### 🔹 Table de Faits
- **fait_ventes** (50 000 transactions)

### 🔹 Tables de Dimensions
- **dim_temps** (Calendrier 2023–2025)
- **dim_produit** (50 produits, 7 catégories)
- **dim_client** (5 000 clients)
- **dim_magasin** (5 magasins en France)

---

## 🛍️ Catégories Produits

- Fruits & Légumes  
- Viandes & Poissons  
- Produits laitiers  
- Épicerie  
- Boissons  
- Hygiène & Beauté  
- Entretien  

---

## 👥 Segmentation Client

- Tranches d’âge (18–25, 26–35, etc.)
- Statut fidélité : Bronze, Silver, Gold, Platinum
- Segment comportemental : Occasionnel, Régulier, VIP

---

## 💰 Règles de Remise Fidélité

| Statut | Remise |
|--------|--------|
| Bronze | 0% |
| Silver | 5% |
| Gold | 10% |
| Platinum | 15% |

---

# ⚙️ Pipeline ETL

## 1️⃣ Extraction
- Génération des produits
- Génération des clients
- Calendrier 3 ans
- Simulation de 50 000 ventes

## 2️⃣ Transformation
- Nettoyage des valeurs manquantes
- Suppression des doublons
- Validation des types
- Création des tranches d’âge
- Application des remises fidélité
- Vérification intégrité référentielle

## 3️⃣ Chargement
- Chargement des dimensions
- Chargement de la table de faits
- Indexation des clés étrangères

---

# 🔍 Analyses OLAP

## 📌 Top 10 Produits par Chiffre d’Affaires
Permet d’identifier les produits stratégiques  
→ Observation possible de la **loi de Pareto (80/20)**

## 📅 Évolution Mensuelle du Chiffre d’Affaires
Analyse :
- Saisonnalité
- Tendances annuelles
- Panier moyen
- Volume de transactions

---

# 📈 Résultats Attendus

- Vision 360° des ventes
- Identification des produits stars
- Analyse comportementale client
- Optimisation des stocks
- Aide à la prise de décision stratégique

---

# 🎓 Compétences Développées

- Modélisation dimensionnelle
- Conception Data Warehouse
- Développement pipeline ETL
- Optimisation requêtes SQL
- Analyse multidimensionnelle
- Data Visualisation

---

# 🚀 Conclusion

Ce projet met en œuvre une architecture décisionnelle complète permettant :

✔ Une centralisation des données  
✔ Une analyse rapide et performante  
✔ Un support efficace à la décision  

Il constitue une base solide pour évoluer vers :
- BI avancée
- Data Mining
- Machine Learning prédictif






