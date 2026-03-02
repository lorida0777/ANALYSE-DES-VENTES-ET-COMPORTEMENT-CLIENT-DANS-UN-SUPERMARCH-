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

│Composant │Technologie │Justification │
|------------|------------|-------|
│Base de données │PostgreSQL │16SGBD relationnel robuste│
│support OLAPETL│Python 3.x + Pandas │Manipulation de données flexible│
│ORM│ SQLAlchemy│ Abstraction base de données, portabilité│
│Visualisation│ Matplotlib + Seaborn │ Graphiques personnalisables│
│Génération données │Faker + NumPy │Données réalistes pour tests│
│Documentation│Jupyter Notebook │Documentation interactive│


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
       ┌──────────────▼──────────────┐
       │   DATA WAREHOUSE (PostgreSQL) │
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

Dictionnaire de Données
📅 DIM_TEMPS (1 096 enregistrements)
ColonneTypeDescriptiondate_idINTEGER (PK)Identifiant unique de la datedate_completeDATEDate complète (YYYY-MM-DD)jourINTEGERJour du mois (1-31)moisINTEGERMois (1-12)trimestreINTEGERTrimestre (1-4)anneeINTEGERAnnée (2023-2025)jour_semaineINTEGERJour de la semaine (1=Lundi, 7=Dimanche)nom_jourVARCHAR(20)Nom du jour (Lundi, Mardi, ...)semaine_anneeINTEGERNuméro de semaine dans l'annéeest_weekendBOOLEANTRUE si samedi/dimancheest_ferieBOOLEANTRUE si jour férié français
🛍️ DIM_PRODUIT (50 enregistrements)
ColonneTypeDescriptionproduit_idINTEGER (PK)Identifiant unique du produitcode_produitVARCHAR(20)Code article (PRD00001, ...)nom_produitVARCHAR(100)Nom commercialcategorieVARCHAR(50)Catégorie (7 catégories)fournisseurVARCHAR(100)Nom du fournisseurprix_unitaireNUMERIC(10,2)Prix de vente unitaire (€)stock_disponibleINTEGERStock actuel
Catégories : Fruits & Légumes, Viandes & Poissons, Produits laitiers, Épicerie, Boissons, Hygiène & Beauté, Entretien
👥 DIM_CLIENT (5 000 enregistrements)
ColonneTypeDescriptionclient_idINTEGER (PK)Identifiant unique du clientageINTEGERÂge (18-80 ans)tranche_ageVARCHAR(10)Segment d'âge (18-25, 26-35, ...)sexeCHAR(1)M ou Fcode_postalVARCHAR(10)Code postalvilleVARCHAR(100)Ville de résidenceregionVARCHAR(100)Région françaisestatut_fideliteVARCHAR(20)Bronze, Silver, Gold, Platinumdate_inscriptionDATEDate d'adhésion programme fidélitésegment_clientVARCHAR(20)Occasionnel, Régulier, VIP
🏬 DIM_MAGASIN (5 enregistrements)
ColonneTypeDescriptionmagasin_idINTEGER (PK)Identifiant unique du magasinnom_magasinVARCHAR(100)Nom commercialvilleVARCHAR(100)Ville d'implantationregionVARCHAR(100)Régioncode_postalVARCHAR(10)Code postalsurface_m2INTEGERSurface de vente (m²)type_magasinVARCHAR(50)Hypermarché, Supermarché, Proximité
Magasins : Paris, Lyon, Marseille, Toulouse, Bordeaux
💰 FAIT_VENTES (50 000 enregistrements)
ColonneTypeDescriptionvente_idINTEGER (PK)Identifiant unique de la transactiondate_idINTEGER (FK)Référence à dim_tempsproduit_idINTEGER (FK)Référence à dim_produitclient_idINTEGER (FK)Référence à dim_clientmagasin_idINTEGER (FK)Référence à dim_magasinquantiteINTEGERNombre d'unités venduesmontant_unitaireNUMERIC(10,2)Prix unitaire au moment de la ventemontant_totalNUMERIC(10,2)quantite × montant_unitaireremiseNUMERIC(10,2)Remise fidélité appliquéemontant_netNUMERIC(10,2)Montant après remise (€)
Remises fidélité :

Bronze : 0%
Silver : 5%
Gold : 10%
Platinum : 15%


⚙️ Pipeline ETL
1️⃣ EXTRACTION (Extract)
Sources de données

Produits : Génération avec Faker (50 produits, 7 catégories)
Clients : Génération aléatoire (5 000 profils démographiques)
Magasins : 5 points de vente répartis en France
Temps : Calendrier 2023-2025 avec jours fériés français
Ventes : 50 000 transactions simulées avec logique métier

Code d'extraction
pythonimport pandas as pd
from faker import Faker

fake = Faker('fr_FR')

# Exemple : Extraction produits
produits_data = []
for categorie, items in categories.items():
    for item in items:
        produits_data.append({
            'produit_id': produit_id,
            'nom_produit': item,
            'categorie': categorie,
            'prix_unitaire': round(random.uniform(1.5, 25.0), 2)
        })

df_produits = pd.DataFrame(produits_data)
2️⃣ TRANSFORMATION (Transform)
Opérations de transformation
Nettoyage :

Vérification des valeurs manquantes
Détection et suppression des doublons
Validation des types de données

python# Vérification qualité
for name, df in [('Produits', df_produits), ('Clients', df_clients)]:
    missing = df.isnull().sum().sum()
    duplicates = df.duplicated().sum()
    print(f"{name} - Manquants: {missing}, Doublons: {duplicates}")
Enrichissement :

Calcul des tranches d'âge
Ajout de clés de substitution (surrogate keys)
Calcul du panier moyen
Application des règles de remise selon fidélité

python# Enrichissement tranche d'âge
if age < 26:
    tranche_age = '18-25'
elif age < 36:
    tranche_age = '26-35'
# ...

# Calcul remise
remise_pct = {'Bronze': 0, 'Silver': 0.05, 'Gold': 0.10, 'Platinum': 0.15}
remise = montant_total * remise_pct.get(statut_fidelite, 0)
Validation :

Cohérence des dates
Montants positifs
Quantités > 0
Intégrité référentielle

3️⃣ CHARGEMENT (Load)
Stratégie de chargement
Ordre de chargement (respecter les contraintes de clés étrangères) :

Tables de dimensions (dim_temps, dim_produit, dim_client, dim_magasin)
Table de faits (fait_ventes)

python# Création des tables
with engine.connect() as conn:
    for statement in sql_create_tables.split(';'):
        conn.execute(text(statement))
    conn.commit()

# Chargement dimensions
df_temps.to_sql('dim_temps', engine, if_exists='append', index=False)
df_produits.to_sql('dim_produit', engine, if_exists='append', index=False)
df_clients.to_sql('dim_client', engine, if_exists='append', index=False)
df_magasins.to_sql('dim_magasin', engine, if_exists='append', index=False)

# Chargement table de faits
df_ventes.to_sql('fait_ventes', engine, if_exists='append', index=False)
Optimisations
Index créés :
sqlCREATE INDEX idx_ventes_date ON fait_ventes(date_id);
CREATE INDEX idx_ventes_produit ON fait_ventes(produit_id);
CREATE INDEX idx_ventes_client ON fait_ventes(client_id);
CREATE INDEX idx_ventes_magasin ON fait_ventes(magasin_id);
Avantages :

Amélioration des performances de requêtes de 10 à 100×
Optimisation des jointures multidimensionnelles
Support efficace des GROUP BY


🔍 Analyses OLAP
Requêtes Multidimensionnelles Implémentées
1️⃣ Top 10 Produits par Chiffre d'Affaires
Objectif : Identifier les produits stars générant le plus de revenus
sqlSELECT 
    p.nom_produit,
    p.categorie,
    COUNT(v.vente_id) as nb_ventes,
    SUM(v.quantite) as quantite_totale,
    SUM(v.montant_net) as ca_total
FROM fait_ventes v
JOIN dim_produit p ON v.produit_id = p.produit_id
GROUP BY p.nom_produit, p.categorie
ORDER BY ca_total DESC
LIMIT 10
Insight : Loi de Pareto (80/20) - 20% des produits génèrent 80% du CA
2️⃣ Évolution du CA Mensuel
Objectif : Analyser les tendances temporelles et saisonnalité
sqlSELECT 
    t.annee,
    t.mois,
    COUNT(DISTINCT v.vente_id) as nb_transactions,
    SUM(v.montant_net) as ca_mensuel,
    AVG(v.montant_net) as panier_moyen
FROM fait_ventes v
JOIN dim_temps t ON v.date_id = t.date_id
GROUP BY t.annee, t.mois
ORDER BY t.annee, t.mois


