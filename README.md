# Prévision des Ventes

## Structure du projet
```
methodology-ml-project/
├── data/
│   ├── raw/                          # Données brutes (fichier initial)
│   │   └── stores_sales_forecasting.csv
│   ├── interim/                      # Données nettoyées, étapes intermédiaires
│   │   └── stores_sales_forecasting_interim.csv
│   └── processed/                    # Données enrichies avec features avancées
│       └── stores_sales_forecasting_processed.csv
│
├── instructions/
│   ├── instructions.md                # Instructions du projet
│
├── models/
│   ├── classifier_pipeline.joblib    # Pipeline complet de classification (prétraitement + XGBClassifier)
│   └── regressor_pipeline.joblib     # Pipeline complet de régression (prétraitement + XGBRegressor)
│
├── notebooks/
│   └── stores_sales_forecasting.ipynb  # Notebook principal (EDA, features, modélisation)
│
├── reports/
│   ├── eda_report.html               # Profiling exploratoire complet
│   └── final_report.md               # Rapport final avec résultats
│
├── requirements.txt                  # Dépendances Python
└── README.md                         # Documentation du projet
```

### Notes
- L'objectif du projet n'est pas de construire une solution ML complète. Il vise plutôt à montrer les étapes globales de mon raisonnement lors du développement d'un projet ML.
- Le notebook contient toutes les étapes (préparation, EDA, modèles, interprétation). Il est organisé en sections pour suivre facilement les étapes de mon raisonnement. Utilisez la barre latérale dans la barre de menu *View → Table of Contents* pour naviguer entre les sections. Ça devrait donner qqchose comme ceci:

#### Structure du Notebook
```
Exploration des données (EDA)
├── Constat
├── Prochaine étape
├── Constat
├── Décision d'orientation
├── Prochaines étapes
├── Constat
└── Prochaines étapes

1. Modèle de Classification : Prédire la profitabilité d'une commande
├── Feature Engineering et Sélection
│   ├── Constat
│   ├── Analyse des résultats et décisions
│   └── Prochaine étape
├── Importance des variables : Pour explicabilité du modèle
│   └── Constat
└── Conclusion Classification

2. Régression conditionnelle : prédire le montant des ventes profitables
├── Constat
├── Prochaine étape
├── Log-Transform
│   ├── Constat
│   ├── Décision
│   └── Prochaine étape
├── Baseline Model (pred = moyenne des ventes du train set)
│   ├── Constat
│   └── Prochaine étape
├── Tuning XGBoost
│   ├── Constat
│   └── Prochaine étape
├── Features avancés
│   ├── Constat
│   ├── Décision
│   └──Prochaines étapes
├── Cross-Validation
│   ├── Constat
│   └── Prochaine étape
└── Suivi des courbes d’apprentissage et early stopping
    ├── Constat
    └── Early Stopping

Exporter le modèle
└── Test pour charger et utiliser le modèle

Conclusion
```
- Le rapport `reports/final_report.md` condense les résultats et oriente la lecture côté business. Il contient l’ensemble des réponses attendues au document d’instructions dans `instructions/instructions.md`
- Le rapport `reports/eda_report.html` est facultatif mais utile pour explorer rapidement les patterns du dataset.

***À évaluer en priorité :**  
- `notebooks/stores_sales_forecasting.ipynb` : contient tout le code, les visuels, le raisonnement, l’EDA, la classification et la régression.
- `reports/final_report.md` : contient l’ensemble des réponses au document d’instructions.  
  - Ce rapport est structuré selon les questions du document d'instructions et fournit une synthèse des résultats obtenus.

## Workflow

J'ai choisi JupyterLab comme environnement car il permet d’itérer rapidement et de documenter mes choix au fur et à mesure.  
C’est aussi une solution collaborative efficace pour tester et comparer les modèles.  

Une fois un modèle validé, il peut être transféré vers une solution de production comme AWS SageMaker afin d’assurer la scalabilité, le monitoring et la maintenance.

## Installation

Clonez le dépôt et installez les dépendances dans un environnement virtuel dédié :

```bash
git clone https://github.com/davlaj/methodology-ml-project.git
cd methodology-ml-project
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Dépendances système

Pour utiliser XGBoost sur macOS, assurez-vous d’installer `libomp` :

```bash
brew install libomp
```

ydata-profiling et tqdm : nécessitent ipywidgets pour les barres de progression interactives:

```bash
pip install ipywidgets
jupyter nbextension enable --py widgetsnbextension
```

OpenSSL (optionnel, macOS) : pour supprimer certains warnings liés à urllib3 :

```bash
brew install openssl
```

### Lancer le notebook

```bash
jupyter lab
```

Le notebook principal du projet est disponible dans :  
`notebooks/stores_sales_forecasting.ipynb`  
C’est ce fichier qu’il faut exécuter pour reproduire toute la solution (préparation des données, modèles et analyses).

---

## Pipeline & Raisonnement

Le notebook suit une démarche progressive, structurée en étapes clés :

1. **Exploration des données (EDA)**
   - Nettoyage des colonnes inutiles et faible cardinalité.
   - Analyse des distributions (`Sales`, `Profit`), détection de saisonnalité.
   - Rapport interactif généré (`reports/eda_report.html`).

2. **Classification (profitabilité d’une commande)**
   - Sélection et encodage des variables explicatives.
   - Analyse de l’importance des variables pour l’explicabilité.
   - Décision : retenir uniquement les transactions profitables pour la suite.

3. **Régression conditionnelle (ventes profitables uniquement)**
   - Transformation log de la cible pour stabiliser la variance.
   - Modèle baseline (moyenne du train set) pour comparaison.
   - Modèle principal : `XGBRegressor` tuné avec `GridSearchCV`.
   - Ajout et test de features avancés (cyclicité, moyennes groupées, high discount) → non retenus car peu informatifs.
   - Cross-validation (5-fold) avec métriques en échelle réelle (MAE, RMSE).
   - Suivi des courbes d’apprentissage + `early stopping`.

4. **Exportation du modèle**
   - Sauvegarde du pipeline complet (préprocessing + modèle).
   - Test de rechargement et prédictions cohérentes.

---

## Résultats

- **RMSE moyen (réel, CV)** : ~419$  
- **MAE moyen (réel, CV)** : ~197$  
- Le modèle XGBoost tuné améliore nettement la performance par rapport au baseline (≈40% de réduction d’erreur).  
- Il prédit correctement les ventes dans la majorité des cas, même si les valeurs extrêmes restent difficiles à estimer.  
- Bien qu’il puisse être amélioré, l'approche en deux étapes permet déjà d'orienter les décisions stratégiques des gestionnaires quant aux rabais, à l'inventaire des produits et aux questions logistiques géographiques.
--------

